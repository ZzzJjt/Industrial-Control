#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>
#include <ctime>

// Matrix class for state-space operations
struct Matrix {
    std::vector<std::vector<double>> data;
    int rows, cols;

    Matrix(int r, int c) : rows(r), cols(c), data(r, std::vector<double>(c, 0.0)) {}

    Matrix operator*(const Matrix& other) const {
        Matrix result(rows, other.cols);
        for (int i = 0; i < rows; ++i)
            for (int j = 0; j < other.cols; ++j)
                for (int k = 0; k < cols; ++k)
                    result.data[i][j] += data[i][k] * other.data[k][j];
        return result;
    }

    Matrix operator+(const Matrix& other) const {
        Matrix result(rows, cols);
        for (int i = 0; i < rows; ++i)
            for (int j = 0; j < cols; ++j)
                result.data[i][j] = data[i][j] + other.data[j][i];
        return result;
    }

    Matrix operator-(const Matrix& other) const {
        Matrix result(rows, cols);
        for (int i = 0; i < rows; ++i)
            for (int j = 0; j < cols; ++j)
                result.data[i][j] = data[i][j] - other.data[j][i];
        return result;
    }
};

// Wind turbine and storage model
struct WindFarmModel {
    // State-space: x = [omega; P; SoC], u = [beta; Ps], d = [v; D]
    Matrix A, B, E, C; // System matrices
    double Ts = 10.0;  // Sampling time (s)
    int nx = 3, nu = 2, nd = 2, ny = 3;
    double P_rated = 5.0; // Rated power (MW)
    double v_nominal = 12.0; // Nominal wind speed (m/s)
    double eta = 0.95; // Storage efficiency

    WindFarmModel() : A(3, 3), B(3, 2), E(3, 2), C(3, 3) {
        // A: Dynamics (rotor inertia, power, SoC)
        A.data = {{-0.1, 0.0, 0.0},
                  {0.0, -0.05, 0.0},
                  {0.0, 0.0, 0.0}}; // SoC integrates Ps
        // B: Control effects (beta, Ps)
        B.data = {{-0.5, 0.0},  // Beta reduces omega
                  {0.2, 0.0},   // Beta affects power
                  {0.0, 1.0}};  // Ps affects SoC
        // E: Disturbance effects (wind speed, demand)
        E.data = {{0.1, 0.0},   // Wind increases omega
                  {0.05, -0.1}, // Wind and demand affect power
                  {0.0, 0.0}};  // No direct effect on SoC
        // C: Output matrix (omega, P, SoC)
        C.data = {{1.0, 0.0, 0.0},
                  {0.0, 1.0, 0.0},
                  {0.0, 0.0, 1.0}};
    }

    Matrix simulate(const Matrix& x, const Matrix& u, const Matrix& d) {
        // Nonlinear dynamics: x(k+1) = f(x(k), u(k), d(k))
        double omega = x.data[0][0];
        double P = x.data[1][0];
        double SoC = x.data[2][0];
        double beta = u.data[0][0];
        double Ps = u.data[1][0];
        double v = d.data[0][0];
        double D = d.data[1][0];

        // Turbine power (simplified Betz law)
        double Cp = 0.5 * (1 - 0.022 * beta * beta); // Power coefficient
        double P_wind = 0.5 * 0.001225 * 1000 * Cp * v * v * v; // MW
        P_wind = std::min(P_wind, P_rated);

        // Dynamics
        double d_omega = -0.1 * omega + 0.1 * v - 0.5 * beta;
        double d_P = -0.05 * P + 0.2 * P_wind - 0.1 * D;
        double d_SoC = Ps / (10.0 * eta); // 10 MWh capacity, Ps in MW

        Matrix x_next(3, 1);
        x_next.data[0][0] = omega + Ts * d_omega;
        x_next.data[1][0] = P + Ts * d_P;
        x_next.data[2][0] = SoC + Ts * d_SoC;

        // Apply constraints
        x_next.data[0][0] = std::max(5.0, std::min(20.0, x_next.data[0][0])); // omega
        x_next.data[1][0] = std::max(0.0, std::min(5.0, x_next.data[1][0]));  // P
        x_next.data[2][0] = std::max(10.0, std::min(90.0, x_next.data[2][0])); // SoC
        return x_next;
    }
};

// MPC Controller
struct MPCController {
    WindFarmModel model;
    int Np = 10; // Prediction horizon
    int Nc = 3;  // Control horizon
    double Q[3] = {1.0, 10.0, 5.0}; // Weights: omega, P, SoC
    double R[2] = {0.1, 0.2};       // Weights: beta, Ps (penalize Ps for efficiency)
    double S[2] = {0.01, 0.02};     // Weights: d_beta, d_Ps
    double omega_min = 5.0, omega_max = 20.0;
    double P_min = 0.0, P_max = 5.0;
    double SoC_min = 10.0, SoC_max = 90.0;
    double beta_min = 0.0, beta_max = 30.0;
    double Ps_min = -2.0, Ps_max = 2.0; // Charge/discharge limits (MW)
    double dbeta_max = 1.0 * model.Ts;  // Max pitch rate (deg/s)
    double dPs_max = 0.5 * model.Ts;    // Max charge/discharge rate (MW/s)

    double objective(const Matrix& u, const Matrix& x0, const std::vector<std::vector<double>>& ref,
                     const std::vector<double>& v, const std::vector<double>& D, int k) {
        Matrix u_traj(Nc * model.nu, 1);
        for (int i = 0; i < Nc * model.nu; ++i)
            u_traj.data[i][0] = u.data[i][0];
        
        Matrix x = x0;
        double J = 0.0;
        
        // Predict states
        for (int i = 0; i < Np; ++i) {
            Matrix u_i(model.nu, 1);
            int idx = std::min(i, Nc-1) * model.nu;
            u_i.data[0][0] = u_traj.data[idx][0];
            u_i.data[1][0] = u_traj.data[idx+1][0];
            
            Matrix d(model.nd, 1);
            d.data[0][0] = v[std::min(k + i, (int)v.size() - 1)];
            d.data[1][0] = D[std::min(k + i, (int)D.size() - 1)];
            
            Matrix y = model.C * x;
            std::vector<double> r = ref[std::min(k + i, (int)ref.size() - 1)];
            double e_omega = y.data[0][0] - r[0];
            double e_P = y.data[1][0] - r[1];
            double e_SoC = y.data[2][0] - r[2];
            J += Q[0] * e_omega * e_omega + Q[1] * e_P * e_P + Q[2] * e_SoC * e_SoC;
            
            if (i < Nc) {
                J += R[0] * u_i.data[0][0] * u_i.data[0][0] + 
                     R[1] * u_i.data[1][0] * u_i.data[1][0];
                if (i > 0) {
                    double d_beta = u_traj.data[idx][0] - u_traj.data[idx-model.nu][0];
                    double d_Ps = u_traj.data[idx+1][0] - u_traj.data[idx-model.nu+1][0];
                    J += S[0] * d_beta * d_beta + S[1] * d_Ps * d_Ps;
                }
            }
            
            x = model.simulate(x, u_i, d);
        }
        
        return J;
    }

    Matrix compute_control(const Matrix& x0, const std::vector<std::vector<double>>& ref,
                          const std::vector<double>& v, const std::vector<double>& D, int k,
                          const Matrix* u_prev) {
        Matrix u(Nc * model.nu, 1);
        for (int i = 0; i < Nc; ++i) {
            u.data[i * model.nu][0] = 10.0; // Initial beta
            u.data[i * model.nu + 1][0] = 0.0; // Initial Ps
        }
        
        // Gradient descent (simplified QP)
        double alpha = 0.01;
        double J = objective(u, x0, ref, v, D, k);
        for (int iter = 0; iter < 100; ++iter) {
            Matrix grad(Nc * model.nu, 1);
            for (int i = 0; i < Nc * model.nu; ++i) {
                double u_orig = u.data[i][0];
                u.data[i][0] += 1e-5;
                double J_new = objective(u, x0, ref, v, D, k);
                grad.data[i][0] = (J_new - J) / 1e-5;
                u.data[i][0] = u_orig;
            }
            
            for (int i = 0; i < Nc * model.nu; ++i) {
                u.data[i][0] -= alpha * grad.data[i][0];
                if (i % 2 == 0)
                    u.data[i][0] = std::max(beta_min, std::min(beta_max, u.data[i][0]));
                else
                    u.data[i][0] = std::max(Ps_min, std::min(Ps_max, u.data[i][0]));
                
                // Rate constraints
                if (u_prev && i < 2) {
                    double d_u = u.data[i][0] - (*u_prev).data[i/2][0];
                    double d_max = (i % 2 == 0) ? dbeta_max : dPs_max;
                    u.data[i][0] = (*u_prev).data[i/2][0] + std::max(-d_max, std::min(d_max, d_u));
                }
            }
            
            J = objective(u, x0, ref, v, D, k);
        }
        
        Matrix u_out(model.nu, 1);
        u_out.data[0][0] = u.data[0][0];
        u_out.data[1][0] = u.data[1][0];
        return u_out;
    }
};

int main() {
    WindFarmModel model;
    MPCController mpc;
    srand(static_cast<unsigned>(time(0)));

    // Simulation parameters
    int n_sim = 360; // 3600s (1 hour)
    Matrix x(3, 1);  // State: [omega; P; SoC]
    x.data = {{10.0}, {3.0}, {50.0}}; // Initial: 10 rad/s, 3 MW, 50%
    std::vector<std::vector<double>> ref(n_sim, {10.0, 4.0, 50.0}); // Setpoint: omega, P, SoC
    std::vector<std::string> scenarios = {"Steady", "Gusty", "LoadSurge"};

    for (const auto& scenario : scenarios) {
        std::cout << "\nScenario: " << scenario << "\n";
        
        // Generate wind speed and demand
        std::vector<double> v(n_sim, 12.0); // Nominal 12 m/s
        std::vector<double> D(n_sim, 4.0);  // Nominal 4 MW
        if (scenario == "Steady") {
            // Constant conditions
        } else if (scenario == "Gusty") {
            for (int i = 0; i < n_sim; ++i)
                v[i] += (rand() % 1000) / 100.0 - 5.0; // ±5 m/s noise
        } else if (scenario == "LoadSurge") {
            for (int i = 120; i < 240; ++i)
                D[i] = 4.5; // Surge to 4.5 MW at 1200–2400s
        }
        v = std::vector<double>(v.begin(), v.begin() + n_sim);
        D = std::vector<double>(D.begin(), D.begin() + n_sim);
        for (int i = 0; i < n_sim; ++i) {
            v[i] = std::max(0.0, std::min(25.0, v[i]));
            D[i] = std::max(0.0, std::min(5.0, D[i]));
            ref[i][1] = D[i]; // Track demand
        }

        // Simulate
        std::vector<std::vector<double>> x_traj(n_sim, std::vector<double>(3));
        std::vector<std::vector<double>> u_traj(n_sim, std::vector<double>(2));
        std::vector<double> P_grid(n_sim);
        x_traj[0] = {x.data[0][0], x.data[1][0], x.data[2][0]};
        u_traj[0] = {10.0, 0.0};
        double energy = 0.0;
        int violations = 0;

        for (int k = 0; k < n_sim-1; ++k) {
            Matrix x_k(3, 1);
            x_k.data = {{x_traj[k][0]}, {x_traj[k][1]}, {x_traj[k][2]}};
            Matrix u_prev(2, 1);
            u_prev.data = {{u_traj[k][0]}, {u_traj[k][1]}};

            Matrix u = mpc.compute_control(x_k, ref, v, D, k, &u_prev);
            u_traj[k+1] = {u.data[0][0], u.data[1][0]};

            Matrix d(2, 1);
            d.data = {{v[k]}, {D[k]}};
            x = model.simulate(x_k, u, d);
            x_traj[k+1] = {x.data[0][0], x.data[1][0], x.data[2][0]};

            // Grid power: P - Ps (Ps positive for charging, negative for discharging)
            P_grid[k] = x_traj[k][1] - u_traj[k][1];
            energy += u_traj[k][1] * model.Ts / 3600; // Fuel equivalent (kg)

            // Check constraints
            if (x_traj[k][0] < mpc.omega_min || x_traj[k][0] > mpc.omega_max ||
                x_traj[k][1] < mpc.P_min || x_traj[k][1] > mpc.P_max ||
                x_traj[k][2] < mpc.SoC_min || x_traj[k][2] > mpc.SoC_max)
                violations++;
        }
        P_grid[n_sim-1] = x_traj[n_sim-1][1] - u_traj[n_sim-1][1];

        // Metrics
        double iae_omega = 0.0, iae_P = 0.0, iae_SoC = 0.0;
        double grid_stability = 0.0;
        for (int k = 0; k < n_sim; ++k) {
            iae_omega += std::abs(x_traj[k][0] - ref[k][0]) * model.Ts;
            iae_P += std::abs(x_traj[k][1] - ref[k][1]) * model.Ts;
            iae_SoC += std::abs(x_traj[k][2] - ref[k][2]) * model.Ts;
            grid_stability += std::abs(P_grid[k] - D[k]) * model.Ts;
        }

        // Output results
        std::cout << "IAE: omega=" << iae_omega << "rad·s, P=" << iae_P << "MW·s, SoC=" << iae_SoC << "%·s\n";
        std::cout << "Energy Consumption (Fuel Equivalent): " << energy << "kg\n";
        std::cout << "Grid Stability (IAE): " << grid_stability << "MW·s\n";
        std::cout << "Constraint Violations: " << violations << "\n";
        std::cout << "Sample Outputs (every 600s):\n";
        std::cout << "Time(s) | omega(rad/s) | P(MW) | SoC(%) | beta(deg) | Ps(MW) | P_grid(MW)\n";
        for (int k = 0; k < n_sim; k += 60) {
            printf("%7.1f | %12.2f | %5.2f | %6.2f | %9.2f | %6.2f | %10.2f\n",
                   k * model.Ts, x_traj[k][0], x_traj[k][1], x_traj[k][2],
                   u_traj[k][0], u_traj[k][1], P_grid[k]);
        }
    }

    return 0;
}
