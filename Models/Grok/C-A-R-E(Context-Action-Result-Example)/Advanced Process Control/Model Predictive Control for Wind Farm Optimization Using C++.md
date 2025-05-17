#include <vector>
#include <cmath>
#include <algorithm>
#include <iostream>

// Wind Turbine and BESS Model
class WindFarm {
public:
    double dt; // Time step (s)
    std::vector<double> omega; // Rotor speeds (rad/s)
    double SOC; // Battery state of charge (%)
    double J; // Moment of inertia (kg·m²)
    double rho; // Air density (kg/m³)
    double A; // Rotor area (m²)
    double C_T; // BESS capacitance (MW·h/%)
    double P_nom; // Nominal turbine power (MW)

    WindFarm(double dt_ = 10.0) : dt(dt_), omega({15.0, 15.0, 15.0}), SOC(50.0),
        J(1e6), rho(1.225), A(5000.0), C_T(0.01), P_nom(5.0) {}

    double compute_Cp(double beta, double lambda) {
        // Simplified power coefficient (nonlinear)
        double c1 = 0.5, c2 = 116.0, c3 = 0.4, c4 = 5.0, c5 = 21.0;
        double x = 1.0 / (1.0 / (lambda + 0.08 * beta) - 0.035 / (beta * beta * beta + 1));
        return c1 * (c2 / x - c3 * beta - c4) * std::exp(-c5 / x);
    }

    std::vector<double> step(const std::vector<double>& u, double v_w) {
        // u = [beta1, beta2, beta3, P_batt]
        std::vector<double> P(3);
        for (int i = 0; i < 3; ++i) {
            double beta = u[i];
            double lambda = omega[i] * 50.0 / v_w; // Tip-speed ratio (R=50m)
            double Cp = compute_Cp(beta, lambda);
            // Power: P = 0.5*rho*A*Cp*v_w^3 / 1e6 (MW)
            P[i] = 0.5 * rho * A * Cp * std::pow(v_w, 3) / 1e6;
            // Dynamics: d(omega)/dt = (P*1e6/omega - D*omega)/J
            double torque = P[i] * 1e6 / omega[i];
            double D = 100.0; // Damping
            double domega_dt = (torque - D * omega[i]) / J;
            omega[i] += domega_dt * dt;
            omega[i] = std::clamp(omega[i], 10.0, 20.0); // Speed constraints
        }
        // BESS: d(SOC)/dt = -P_batt/C_T
        double P_batt = u[3];
        SOC += (-P_batt / C_T) * dt / 3600.0; // Convert to %/s
        SOC = std::clamp(SOC, 20.0, 80.0); // SOC constraints
        return P;
    }
};

// PID Controller (Baseline)
class PID {
public:
    double Kp, Ki, dt;
    double integral, prev_error;

    PID(double Kp_ = 0.5, double Ki_ = 0.01, double dt_ = 10.0)
        : Kp(Kp_), Ki(Ki_), dt(dt_), integral(0.0), prev_error(0.0) {}

    double compute(double setpoint, double pv) {
        double error = setpoint - pv;
        integral += error * dt;
        double u = Kp * error + Ki * integral;
        prev_error = error;
        return std::clamp(u, 0.0, 20.0); // Beta limit
    }
};

// MPC Controller
class MPC {
public:
    WindFarm& farm;
    int horizon; // Prediction horizon (steps)
    int control_horizon; // Control horizon (steps)
    double dt; // Time step (s)
    std::vector<double> Q; // Weights on [omega1, omega2, omega3, SOC] errors
    std::vector<double> R; // Weights on [beta1, beta2, beta3, P_batt] effort
    double omega_setpoint; // rad/s
    double P_demand; // MW
    std::vector<std::vector<double>> A; // Linearized state matrix
    std::vector<std::vector<double>> B; // Linearized input matrix

    MPC(WindFarm& f, int h = 10, int ch = 3)
        : farm(f), horizon(h), control_horizon(ch), dt(f.dt),
          Q({1.0, 1.0, 1.0, 0.1}), R({0.1, 0.1, 0.1, 0.01}),
          omega_setpoint(15.0), P_demand(15.0) {
        // Linearized model at nominal: omega=15, beta=10, v_w=12, SOC=50, P_batt=0
        A = {{1.0, 0.0, 0.0, 0.0},
             {0.0, 1.0, 0.0, 0.0},
             {0.0, 0.0, 1.0, 0.0},
             {0.0, 0.0, 0.0, 1.0 - dt / (3600.0 * f.C_T)}};
        B = {{f.P_nom * 0.01 * dt / f.J, 0.0, 0.0, 0.0},
             {0.0, f.P_nom * 0.01 * dt / f.J, 0.0, 0.0},
             {0.0, 0.0, f.P_nom * 0.01 * dt / f.J, 0.0},
             {0.0, 0.0, 0.0, -dt / (3600.0 * f.C_T)}};
    }

    std::vector<double> predict(const std::vector<double>& state,
                               const std::vector<double>& u_traj,
                               const std::vector<double>& v_w_future) {
        std::vector<double> states((horizon + 1) * 4);
        for (int i = 0; i < 4; ++i) states[i] = state[i];
        std::vector<double> x = state;
        for (int k = 0; k < horizon; ++k) {
            int u_idx = std::min(k, control_horizon - 1) * 4;
            std::vector<double> u = {u_traj[u_idx], u_traj[u_idx + 1], 
                                    u_traj[u_idx + 2], u_traj[u_idx + 3]};
            // Approximate power for linear model
            double P = 0.5 * farm.rho * farm.A * 0.3 * std::pow(v_w_future[k], 3) / 1e6;
            std::vector<double> d = {P / farm.J * dt, P / farm.J * dt, P / farm.J * dt, 0.0};
            for (int i = 0; i < 4; ++i) {
                x[i] = 0.0;
                for (int j = 0; j < 4; ++j) x[i] += A[i][j] * states[k * 4 + j];
                x[i] += B[i][0] * u[0] + B[i][1] * u[1] + B[i][2] * u[2] + 
                        B[i][3] * u[3] + d[i];
            }
            x[0] = std::clamp(x[0], 10.0, 20.0);
            x[1] = std::clamp(x[1], 10.0, 20.0);
            x[2] = std::clamp(x[2], 10.0, 20.0);
            x[3] = std::clamp(x[3], 20.0, 80.0);
            for (int i = 0; i < 4; ++i) states[(k + 1) * 4 + i] = x[i];
        }
        return states;
    }

    double objective(const std::vector<double>& u_traj, const std::vector<double>& state,
                    const std::vector<double>& v_w_future, const std::vector<double>& u_prev) {
        auto states = predict(state, u_traj, v_w_future);
        double J = 0.0;
        // Cost: power tracking + state error + control effort
        for (int k = 0; k <= horizon; ++k) {
            double P_total = 0.0;
            for (int i = 0; i < 3; ++i) {
                double lambda = states[k * 4 + i] * 50.0 / v_w_future[k];
                double beta = u_traj[std::min(k, control_horizon - 1) * 4 + i];
                double Cp = farm.compute_Cp(beta, lambda);
                P_total += 0.5 * farm.rho * farm.A * Cp * std::pow(v_w_future[k], 3) / 1e6;
            }
            P_total += u_traj[std::min(k, control_horizon - 1) * 4 + 3]; // P_batt
            J += 10.0 * std::pow(P_total - P_demand, 2);
            for (int i = 0; i < 3; ++i)
                J += Q[i] * std::pow(states[k * 4 + i] - omega_setpoint, 2);
            J += Q[3] * std::pow(states[k * 4 + 3] - 50.0, 2); // SOC target
        }
        for (int k = 0; k < control_horizon; ++k) {
            for (int i = 0; i < 4; ++i) J += R[i] * std::pow(u_traj[k * 4 + i], 2);
            // Rate penalty
            for (int i = 0; i < 4; ++i) {
                double d = u_traj[k * 4 + i] - (k == 0 ? u_prev[i] : u_traj[(k-1)*4+i]);
                J += 0.1 * d * d;
            }
        }
        // Constraint penalties
        for (int k = 0; k <= horizon; ++k) {
            for (int i = 0; i < 3; ++i) {
                if (states[k * 4 + i] < 10.0) J += 1e5 * std::pow(10.0 - states[k * 4 + i], 2);
                if (states[k * 4 + i] > 20.0) J += 1e5 * std::pow(states[k * 4 + i] - 20.0, 2);
            }
            if (states[k * 4 + 3] < 20.0) J += 1e5 * std::pow(20.0 - states[k * 4 + 3], 2);
            if (states[k * 4 + 3] > 80.0) J += 1e5 * std::pow(states[k * 4 + 3] - 80.0, 2);
        }
        return J;
    }

    std::vector<double> compute(const std::vector<double>& state,
                               const std::vector<double>& v_w_future,
                               const std::vector<double>& u_prev) {
        std::vector<double> u_traj(control_horizon * 4, 0.0);
        for (int i = 0; i < control_horizon; ++i) {
            u_traj[i * 4] = 10.0; // beta1
            u_traj[i * 4 + 1] = 10.0; // beta2
            u_traj[i * 4 + 2] = 10.0; // beta3
            u_traj[i * 4 + 3] = 0.0; // P_batt
        }
        std::vector<double> u_best = u_traj;
        double J_best = objective(u_traj, state, v_w_future, u_prev);
        double step = 1.0;
        const int max_iter = 100;

        for (int iter = 0; iter < max_iter; ++iter) {
            std::vector<double> grad(u_traj.size(), 0.0);
            double eps = 1e-3;
            for (size_t i = 0; i < u_traj.size(); ++i) {
                std::vector<double> u_plus = u_traj, u_minus = u_traj;
                u_plus[i] += eps;
                u_minus[i] -= eps;
                grad[i] = (objective(u_plus, state, v_w_future, u_prev) -
                          objective(u_minus, state, v_w_future, u_prev)) / (2 * eps);
            }
            for (size_t i = 0; i < u_traj.size(); ++i) {
                u_traj[i] -= step * grad[i];
                u_traj[i] = std::clamp(u_traj[i], i % 4 == 3 ? -5.0 : 0.0, i % 4 == 3 ? 5.0 : 20.0);
            }
            double J = objective(u_traj, state, v_w_future, u_prev);
            if (J < J_best) {
                J_best = J;
                u_best = u_traj;
                step *= 1.1;
            } else {
                u_traj = u_best;
                step *= 0.5;
            }
        }
        // Apply rate constraints
        for (int i = 0; i < 3; ++i)
            u_best[i] = std::clamp(u_best[i], u_prev[i] - 1.0, u_prev[i] + 1.0);
        u_best[3] = std::clamp(u_best[3], u_prev[3] - 1.0, u_prev[3] + 1.0);
        return {u_best[0], u_best[1], u_best[2], u_best[3]};
    }
};

// Simulation
void simulate(WindFarm& farm, MPC& mpc, PID& pid, double t_sim = 3600.0, double dt = 10.0) {
    int n_steps = static_cast<int>(t_sim / dt);
    std::vector<double> t(n_steps);
    std::vector<std::vector<double>> omega_mpc(n_steps, std::vector<double>(3));
    std::vector<double> SOC_mpc(n_steps), P_total_mpc(n_steps);
    std::vector<std::vector<double>> u_mpc(n_steps, std::vector<double>(4));
    std::vector<std::vector<double>> omega_pid(n_steps, std::vector<double>(3));
    std::vector<double> SOC_pid(n_steps), P_total_pid(n_steps);
    std::vector<std::vector<double>> u_pid(n_steps, std::vector<double>(4));
    std::vector<double> v_w(n_steps);
    
    // Wind speed profile
    for (int i = 0; i < n_steps; ++i) {
        t[i] = i * dt;
        v_w[i] = (t[i] < 1800) ? 12.0 : 8.0; // Drop at t=1800s
    }
    
    // MPC simulation
    WindFarm farm_mpc = farm;
    std::vector<double> u_prev = {10.0, 10.0, 10.0, 0.0}; // [beta1, beta2, beta3, P_batt]
    for (int i = 0; i < n_steps; ++i) {
        std::vector<double> state = {farm_mpc.omega[0], farm_mpc.omega[1], 
                                    farm_mpc.omega[2], farm_mpc.SOC};
        std::vector<double> v_w_future(horizon);
        for (int j = 0; j < horizon; ++j) {
            int idx = std::min(i + j, n_steps - 1);
            v_w_future[j] = v_w[idx];
        }
        auto u = mpc.compute(state, v_w_future, u_prev);
        auto P = farm_mpc.step(u, v_w[i]);
        omega_mpc[i] = farm_mpc.omega;
        SOC_mpc[i] = farm_mpc.SOC;
        P_total_mpc[i] = std::accumulate(P.begin(), P.end(), 0.0) + u[3];
        u_mpc[i] = u;
        u_prev = u;
    }
    
    // PID simulation (no BESS control)
    WindFarm farm_pid = farm;
    for (int i = 0; i < n_steps; ++i) {
        std::vector<double> u(4, 0.0);
        for (int j = 0; j < 3; ++j)
            u[j] = pid.compute(mpc.omega_setpoint, farm_pid.omega[j]);
        u[3] = 0.0; // No BESS control
        auto P = farm_pid.step(u, v_w[i]);
        omega_pid[i] = farm_pid.omega;
        SOC_pid[i] = farm_pid.SOC;
        P_total_pid[i] = std::accumulate(P.begin(), P.end(), 0.0);
        u_pid[i] = u;
    }
    
    // Calculate performance metrics
    double mse_P_mpc = 0.0, mse_P_pid = 0.0, energy_mpc = 0.0, energy_pid = 0.0;
    for (int i = 0; i < n_steps; ++i) {
        mse_P_mpc += std::pow(P_total_mpc[i] - mpc.P_demand, 2);
        mse_P_pid += std::pow(P_total_pid[i] - mpc.P_demand, 2);
        for (int j = 0; j < 3; ++j) energy_mpc += u_mpc[i][j];
        energy_mpc += std::abs(u_mpc[i][3]);
        for (int j = 0; j < 3; ++j) energy_pid += u_pid[i][j];
    }
    mse_P_mpc /= n_steps;
    mse_P_pid /= n_steps;
    energy_mpc *= dt / 3600.0; // MJ
    energy_pid *= dt / 3600.0; // MJ
    
    std::cout << "MPC Power MSE: " << mse_P_mpc << " MW²\n";
    std::cout << "PID Power MSE: " << mse_P_pid << " MW²\n";
    std::cout << "MPC Energy: " << energy_mpc << " MJ\n";
    std::cout << "PID Energy: " << energy_pid << " MJ\n";
    
    // Store results (no visualization)
    // Results in omega_mpc, SOC_mpc, P_total_mpc, u_mpc, omega_pid, SOC_pid, P_total_pid, u_pid
}

int main() {
    WindFarm farm(10.0); // 10s time step
    MPC mpc(farm, 10, 3); // 10-step horizon, 3-step control horizon
    PID pid(0.5, 0.01, 10.0); // PID for each turbine
    simulate(farm, mpc, pid, 3600.0, 10.0);
    return 0;
}
