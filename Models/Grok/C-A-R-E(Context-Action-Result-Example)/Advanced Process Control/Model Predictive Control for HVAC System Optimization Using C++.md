#include <vector>
#include <cmath>
#include <algorithm>
#include <iostream>

// Building Model
class Building {
public:
    double dt; // Time step (s)
    double T_in; // Indoor temperature (°C)
    double H_in; // Indoor humidity (%)
    double C_T; // Thermal capacitance (kJ/°C)
    double C_H; // Humidity capacitance (g/%)
    double R_T; // Thermal resistance (°C/kW)
    double Q_occ; // Heat load per occupant (kW/person)
    double H_occ; // Humidity load per occupant (g/s/person)

    Building(double dt_ = 60.0) : dt(dt_), T_in(22.0), H_in(50.0),
        C_T(1000.0), C_H(1000.0), R_T(0.05), Q_occ(0.1), H_occ(0.01) {}

    void step(double u_T, double u_H, double T_ext, double occ) {
        // Dynamics: dT/dt = (u_T - (T_in - T_ext)/R_T + Q_occ*occ)/C_T
        double dT_dt = (u_T - (T_in - T_ext) / R_T + Q_occ * occ) / C_T;
        // dH/dt = (u_H + H_occ*occ)/C_H
        double dH_dt = (u_H + H_occ * occ) / C_H;
        T_in += dT_dt * dt;
        H_in += dH_dt * dt;
        T_in = std::clamp(T_in, 18.0, 26.0); // Constraints
        H_in = std::clamp(H_in, 40.0, 60.0);
    }
};

// PID Controller
class PID {
public:
    double Kp, Ki, dt;
    double integral, prev_error;

    PID(double Kp_ = 5.0, double Ki_ = 0.01, double dt_ = 60.0)
        : Kp(Kp_), Ki(Ki_), dt(dt_), integral(0.0), prev_error(0.0) {}

    double compute(double setpoint, double pv) {
        double error = setpoint - pv;
        integral += error * dt;
        double u = Kp * error + Ki * integral;
        prev_error = error;
        return std::clamp(u, 0.0, 100.0); // Input limits
    }
};

// MPC Controller
class MPC {
public:
    Building& building;
    int horizon; // Prediction horizon (steps)
    int control_horizon; // Control horizon (steps)
    double dt; // Time step (s)
    double Q_T, Q_H; // Weights on T, H errors
    double R_T, R_H; // Weights on control effort
    double T_setpoint, H_setpoint; // Setpoints (°C, %)

    MPC(Building& b, int h = 10, int ch = 3)
        : building(b), horizon(h), control_horizon(ch), dt(b.dt),
          Q_T(10.0), Q_H(10.0), R_T(0.1), R_H(0.1),
          T_setpoint(22.0), H_setpoint(50.0) {}

    std::vector<double> predict(const std::vector<double>& u_traj,
                               double T_in, double H_in,
                               const std::vector<double>& T_ext,
                               const std::vector<double>& occ) {
        std::vector<double> states(horizon + 1 * 2); // T_in, H_in for each step
        states[0] = T_in;
        states[1] = H_in;
        double T = T_in, H = H_in;
        for (int k = 0; k < horizon; ++k) {
            int u_idx = std::min(k, control_horizon - 1);
            double u_T = u_traj[u_idx * 2];
            double u_H = u_traj[u_idx * 2 + 1];
            // Linear model: dT/dt = (u_T - (T - T_ext)/R_T + Q_occ*occ)/C_T
            double dT_dt = (u_T - (T - T_ext[k]) / building.R_T + 
                           building.Q_occ * occ[k]) / building.C_T;
            double dH_dt = (u_H + building.H_occ * occ[k]) / building.C_H;
            T += dT_dt * dt;
            H += dH_dt * dt;
            T = std::clamp(T, 18.0, 26.0);
            H = std::clamp(H, 40.0, 60.0);
            states[(k + 1) * 2] = T;
            states[(k + 1) * 2 + 1] = H;
        }
        return states;
    }

    double objective(const std::vector<double>& u_traj, double T_in, double H_in,
                     const std::vector<double>& T_ext, const std::vector<double>& occ,
                     const std::vector<double>& u_prev) {
        auto states = predict(u_traj, T_in, H_in, T_ext, occ);
        double J = 0.0;
        // Cost: state errors + control effort + rate penalty
        for (int k = 0; k <= horizon; ++k) {
            J += Q_T * std::pow(states[k * 2] - T_setpoint, 2);
            J += Q_H * std::pow(states[k * 2 + 1] - H_setpoint, 2);
        }
        for (int k = 0; k < control_horizon; ++k) {
            J += R_T * std::pow(u_traj[k * 2], 2);
            J += R_H * std::pow(u_traj[k * 2 + 1], 2);
            // Rate penalty
            double dT = u_traj[k * 2] - (k == 0 ? u_prev[0] : u_traj[(k-1)*2]);
            double dH = u_traj[k * 2 + 1] - (k == 0 ? u_prev[1] : u_traj[(k-1)*2+1]);
            J += 0.1 * (dT * dT + dH * dH);
        }
        // Constraint penalties
        for (int k = 0; k <= horizon; ++k) {
            if (states[k * 2] < 18.0) J += 1e5 * std::pow(18.0 - states[k * 2], 2);
            if (states[k * 2] > 26.0) J += 1e5 * std::pow(states[k * 2] - 26.0, 2);
            if (states[k * 2 + 1] < 40.0) J += 1e5 * std::pow(40.0 - states[k * 2 + 1], 2);
            if (states[k * 2 + 1] > 60.0) J += 1e5 * std::pow(states[k * 2 + 1] - 60.0, 2);
        }
        return J;
    }

    std::vector<double> compute(double T_in, double H_in,
                                const std::vector<double>& T_ext,
                                const std::vector<double>& occ,
                                const std::vector<double>& u_prev) {
        // Simple gradient descent for optimization
        std::vector<double> u_traj(control_horizon * 2, 50.0); // Initial guess
        std::vector<double> u_best = u_traj;
        double J_best = objective(u_traj, T_in, H_in, T_ext, occ, u_prev);
        double step = 1.0;
        const int max_iter = 100;
        
        for (int iter = 0; iter < max_iter; ++iter) {
            std::vector<double> grad(u_traj.size(), 0.0);
            double eps = 1e-3;
            // Numerical gradient
            for (size_t i = 0; i < u_traj.size(); ++i) {
                std::vector<double> u_plus = u_traj, u_minus = u_traj;
                u_plus[i] += eps;
                u_minus[i] -= eps;
                grad[i] = (objective(u_plus, T_in, H_in, T_ext, occ, u_prev) -
                           objective(u_minus, T_in, H_in, T_ext, occ, u_prev)) / (2 * eps);
            }
            // Update trajectory
            for (size_t i = 0; i < u_traj.size(); ++i) {
                u_traj[i] -= step * grad[i];
                u_traj[i] = std::clamp(u_traj[i], 0.0, 100.0); // Input constraints
            }
            double J = objective(u_traj, T_in, H_in, T_ext, occ, u_prev);
            if (J < J_best) {
                J_best = J;
                u_best = u_traj;
                step *= 1.1; // Increase step if improving
            } else {
                u_traj = u_best;
                step *= 0.5; // Reduce step if not improving
            }
        }
        // Apply rate constraints
        u_best[0] = std::clamp(u_best[0], u_prev[0] - 10.0, u_prev[0] + 10.0);
        u_best[1] = std::clamp(u_best[1], u_prev[1] - 10.0, u_prev[1] + 10.0);
        return {u_best[0], u_best[1]}; // First control action
    }
};

// Simulation
void simulate(Building& building, MPC& mpc, PID& pid_T, PID& pid_H,
              double t_sim = 7200.0, double dt = 60.0) {
    int n_steps = static_cast<int>(t_sim / dt);
    std::vector<double> t(n_steps);
    std::vector<double> T_mpc(n_steps), H_mpc(n_steps), uT_mpc(n_steps), uH_mpc(n_steps);
    std::vector<double> T_pid(n_steps), H_pid(n_steps), uT_pid(n_steps), uH_pid(n_steps);
    std::vector<double> T_ext(n_steps), occ(n_steps);
    
    // Disturbance profile
    for (int i = 0; i < n_steps; ++i) {
        t[i] = i * dt;
        T_ext[i] = (t[i] < 3600) ? 25.0 : 30.0; // °C, step at t=3600s
        occ[i] = (t[i] < 3600) ? 10.0 : 20.0; // Occupants, step at t=3600s
    }
    
    // MPC simulation
    Building building_mpc = building;
    std::vector<double> u_prev = {50.0, 50.0}; // Initial u_T, u_H
    for (int i = 0; i < n_steps; ++i) {
        // Forecast T_ext and occ (perfect forecast for simplicity)
        std::vector<double> T_ext_future(mpc.horizon);
        std::vector<double> occ_future(mpc.horizon);
        for (int j = 0; j < mpc.horizon; ++j) {
            int idx = std::min(i + j, n_steps - 1);
            T_ext_future[j] = T_ext[idx];
            occ_future[j] = occ[idx];
        }
        // Compute MPC control
        auto u = mpc.compute(building_mpc.T_in, building_mpc.H_in, T_ext_future, occ_future, u_prev);
        building_mpc.step(u[0], u[1], T_ext[i], occ[i]);
        T_mpc[i] = building_mpc.T_in;
        H_mpc[i] = building_mpc.H_in;
        uT_mpc[i] = u[0];
        uH_mpc[i] = u[1];
        u_prev = u;
    }
    
    // PID simulation
    Building building_pid = building;
    for (int i = 0; i < n_steps; ++i) {
        double u_T = pid_T.compute(mpc.T_setpoint, building_pid.T_in);
        double u_H = pid_H.compute(mpc.H_setpoint, building_pid.H_in);
        building_pid.step(u_T, u_H, T_ext[i], occ[i]);
        T_pid[i] = building_pid.T_in;
        H_pid[i] = building_pid.H_in;
        uT_pid[i] = u_T;
        uH_pid[i] = u_H;
    }
    
    // Calculate performance metrics
    double mse_T_mpc = 0.0, mse_H_mpc = 0.0, mse_T_pid = 0.0, mse_H_pid = 0.0;
    double energy_mpc = 0.0, energy_pid = 0.0;
    for (int i = 0; i < n_steps; ++i) {
        mse_T_mpc += std::pow(T_mpc[i] - mpc.T_setpoint, 2);
        mse_H_mpc += std::pow(H_mpc[i] - mpc.H_setpoint, 2);
        mse_T_pid += std::pow(T_pid[i] - mpc.T_setpoint, 2);
        mse_H_pid += std::pow(H_pid[i] - mpc.H_setpoint, 2);
        energy_mpc += (uT_mpc[i] + uH_mpc[i]) * dt / 3600.0; // kWh
        energy_pid += (uT_pid[i] + uH_pid[i]) * dt / 3600.0; // kWh
    }
    mse_T_mpc /= n_steps;
    mse_H_mpc /= n_steps;
    mse_T_pid /= n_steps;
    mse_H_pid /= n_steps;
    
    // Output metrics
    std::cout << "MPC Temperature MSE: " << mse_T_mpc << " °C²\n";
    std::cout << "MPC Humidity MSE: " << mse_H_mpc << " %²\n";
    std::cout << "PID Temperature MSE: " << mse_T_pid << " °C²\n";
    std::cout << "PID Humidity MSE: " << mse_H_pid << " %²\n";
    std::cout << "MPC Energy: " << energy_mpc << " kWh\n";
    std::cout << "PID Energy: " << energy_pid << " kWh\n";
    
    // Store results (no plotting)
    // Results are in T_mpc, H_mpc, uT_mpc, uH_mpc, T_pid, H_pid, uT_pid, uH_pid
}

int main() {
    Building building(60.0); // 60s time step
    MPC mpc(building, 10, 3); // 10-step horizon, 3-step control horizon
    PID pid_T(5.0, 0.01, 60.0); // Temperature PID
    PID pid_H(5.0, 0.01, 60.0); // Humidity PID
    
    simulate(building, mpc, pid_T, pid_H, 7200.0, 60.0);
    return 0;
}
