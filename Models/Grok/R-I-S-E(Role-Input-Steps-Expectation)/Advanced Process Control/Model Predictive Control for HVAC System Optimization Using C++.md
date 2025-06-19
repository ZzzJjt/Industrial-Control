#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

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

struct HVACModel {
    // State-space: x = [T; H], u = [P; Q], d = [Text; N]
    Matrix A, B, E, C; // System matrices
    double Ts = 0.1;   // Sampling time (hours)
    int nx = 2, nu = 2, nd = 2, ny = 2;

    HVACModel() : A(2, 2), B(2, 2), E(2, 2), C(2, 2) {
        // A: Thermal and humidity decay/coupling
        A.data = {{-0.1, 0.01}, {0.02, -0.05}};
        // B: Control effects (power, humidification)
        B.data = {{0.5, 0.0}, {0.0, 0.2}};
        // E: Disturbance effects (external temp, occupancy)
        E.data = {{0.05, 0.01}, {0.0, 0.005}};
        // C: Output matrix (T, H)
        C.data = {{1.0, 0.0}, {0.0, 1.0}};
    }

    Matrix simulate(const Matrix& x, const Matrix& u, const Matrix& d) {
        // x(k+1) = A*x(k) + B*u(k) + E*d(k)
        Matrix x_next = (A * x) + (B * u) + (E * d);
        return x_next;
    }
};

struct MPCController {
    HVACModel model;
    int Np = 10; // Prediction horizon
    int Nc = 3;  // Control horizon
    double Q[2] = {10.0, 5.0}; // State weights (T, H)
    double R[2] = {0.1, 0.05}; // Input weights (P, Q)
    double T_min = 18.0, T_max = 24.0; // Temperature constraints (°C)
    double H_min = 40.0, H_max = 60.0; // Humidity constraints (%)
    double P_min = 0.0, P_max = 10.0;  // Power constraints (kW)
    double Q_min = 0.0, Q_max = 0.5;   // Humidification constraints (kg/h)

    Matrix compute_control(const Matrix& x, const Matrix& r, const std::vector<double>& Text, 
                          const std::vector<double>& N, int k) {
        // Simplified QP solver (gradient descent for demo)
        Matrix u(Nc * model.nu, 1);
        for (int i = 0; i < Nc * model.nu; ++i)
            u.data[i][0] = 0.5; // Initial guess

        // Prediction
        Matrix x_pred(model.nx, Np);
        x_pred.data[0][0] = x.data[0][0];
        x_pred.data[1][0] = x.data[1][0];
        Matrix u_opt(model.nu, Nc);
        for (int i = 0; i < Nc; ++i)
            for (int j = 0; j < model.nu; ++j)
                u_opt.data[j][i] = u.data[i * model.nu + j][0];

        // Objective: min J = sum((y-r)'Q(y-r) + u'Ru)
        double J = 0.0;
        for (int i = 0; i < Np; ++i) {
            Matrix y = model.C * x_pred;
            double e_T = y.data[0][i] - r.data[0][0];
            double e_H = y.data[1][i] - r.data[1][0];
            J += Q[0] * e_T * e_T + Q[1] * e_H * e_H;
            if (i < Nc) {
                J += R[0] * u_opt.data[0][i] * u_opt.data[0][i] + 
                     R[1] * u_opt.data[1][i] * u_opt.data[1][i];
            }
            // Predict next state
            Matrix d(model.nd, 1);
            d.data[0][0] = Text[std::min(k + i, (int)Text.size() - 1)];
            d.data[1][0] = N[std::min(k + i, (int)N.size() - 1)];
            Matrix u_i(model.nu, 1);
            u_i.data[0][0] = i < Nc ? u_opt.data[0][i] : u_opt.data[0][Nc-1];
            u_i.data[1][0] = i < Nc ? u_opt.data[1][i] : u_opt.data[1][Nc-1];
            x_pred = model.simulate(x_pred, u_i, d);
            for (int j = 0; j < model.nx; ++j)
                x_pred.data[j][std::min(i+1, Np-1)] = x_pred.data[j][i];
        }

        // Gradient descent (simplified QP)
        double alpha = 0.01;
        for (int iter = 0; iter < 100; ++iter) {
            Matrix grad(Nc * model.nu, 1);
            for (int i = 0; i < Nc * model.nu; ++i) {
                double u_orig = u.data[i][0];
                u.data[i][0] += 1e-5;
                double J_new = 0.0;
                x_pred.data[0][0] = x.data[0][0];
                x_pred.data[1][0] = x.data[1][0];
                for (int j = 0; j < Np; ++j) {
                    Matrix y = model.C * x_pred;
                    double e_T = y.data[0][j] - r.data[0][0];
                    double e_H = y.data[1][j] - r.data[1][0];
                    J_new += Q[0] * e_T * e_T + Q[1] * e_H * e_H;
                    if (j < Nc) {
                        J_new += R[0] * u_opt.data[0][j] * u_opt.data[0][j] + 
                                 R[1] * u_opt.data[1][j] * u_opt.data[1][j];
                    }
                    Matrix d(model.nd, 1);
                    d.data[0][0] = Text[std::min(k + j, (int)Text.size() - 1)];
                    d.data[1][0] = N[std::min(k + j, (int)N.size() - 1)];
                    Matrix u_i(model.nu, 1);
                    u_i.data[0][0] = j < Nc ? u_opt.data[0][j] : u_opt.data[0][Nc-1];
                    u_i.data[1][0] = j < Nc ? u_opt.data[1][j] : u_opt.data[1][Nc-1];
                    x_pred = model.simulate(x_pred, u_i, d);
                    for (int m = 0; m < model.nx; ++m)
                        x_pred.data[m][std::min(j+1, Np-1)] = x_pred.data[m][j];
                }
                grad.data[i][0] = (J_new - J) / 1e-5;
                u.data[i][0] = u_orig;
            }
            for (int i = 0; i < Nc * model.nu; ++i) {
                u.data[i][0] -= alpha * grad.data[i][0];
                if (i % 2 == 0)
                    u.data[i][0] = std::max(P_min, std::min(P_max, u.data[i][0]));
                else
                    u.data[i][0] = std::max(Q_min, std::min(Q_max, u.data[i][0]));
            }
            J = objective(u);
        }

        Matrix u_out(model.nu, 1);
        u_out.data[0][0] = u.data[0][0];
        u_out.data[1][0] = u.data[1][0];
        return u_out;
    }

    double objective(const Matrix& u) {
        // Placeholder for objective computation (used in gradient descent)
        return 0.0;
    }
};

int main() {
    HVACModel model;
    MPCController mpc;

    // Simulation parameters
    int n_sim = 240; // 24 hours (0.1h steps)
    Matrix x(2, 1);  // State: [T; H]
    x.data = {{20.0}, {45.0}}; // Initial: 20°C, 45%
    Matrix r(2, 1);  // Setpoint: [T=21°C; H=50%]
    r.data = {{21.0}, {50.0}};

    // Disturbance scenarios
    std::vector<std::vector<double>> Text_scenarios = {
        std::vector<double>(n_sim, 20.0), // Typical: 20°C
        std::vector<double>(n_sim, 30.0), // Hot: 30°C
        std::vector<double>(n_sim, 15.0)  // Mild: 15°C
    };
    std::vector<std::vector<double>> N_scenarios = {
        std::vector<double>(n_sim, 20.0), // Typical: 20 people
        std::vector<double>(n_sim, 40.0), // High: 40 people
        std::vector<double>(n_sim, 10.0)  // Low: 10 people
    };
    Text_scenarios[0][60:120] = std::vector<double>(60, 25.0); // Typical: Spike to 25°C at 6–12h
    N_scenarios[0][120:180] = std::vector<double>(60, 30.0);   // Typical: Spike to 30 people at 12–18h

    std::vector<std::string> scenario_names = {"Typical", "Hot_HighOccupancy", "Mild_LowOccupancy"};

    // Simulation loop
    for (size_t s = 0; s < scenario_names.size(); ++s) {
        std::vector<double>& Text = Text_scenarios[s];
        std::vector<double>& N = N_scenarios[s];
        std::vector<double> T_traj(n_sim), H_traj(n_sim), P_traj(n_sim), Q_traj(n_sim);
        double energy = 0.0;
        int violations = 0;
        Matrix x_current = x;

        for (int k = 0; k < n_sim; ++k) {
            // Compute control
            Matrix u = mpc.compute_control(x_current, r, Text, N, k);
            T_traj[k] = x_current.data[0][0];
            H_traj[k] = x_current.data[1][0];
            P_traj[k] = u.data[0][0];
            Q_traj[k] = u.data[1][0];
            energy += P_traj[k] * mpc.model.Ts; // Energy (kWh)
            
            // Check constraints
            if (T_traj[k] < mpc.T_min || T_traj[k] > mpc.T_max ||
                H_traj[k] < mpc.H_min || H_traj[k] > mpc.H_max)
                violations++;

            // Update state
            Matrix d(2, 1);
            d.data[0][0] = Text[k];
            d.data[1][0] = N[k];
            x_current = mpc.model.simulate(x_current, u, d);
        }

        // Performance metrics
        double iae_T = 0.0, iae_H = 0.0;
        for (int k = 0; k < n_sim; ++k) {
            iae_T += std::abs(T_traj[k] - r.data[0][0]) * mpc.model.Ts;
            iae_H += std::abs(H_traj[k] - r.data[1][0]) * mpc.model.Ts;
        }

        // Output results
        std::cout << "\nScenario: " << scenario_names[s] << "\n";
        std::cout << "IAE Temperature: " << iae_T << " °C·h\n";
        std::cout << "IAE Humidity: " << iae_H << " %·h\n";
        std::cout << "Energy Consumption: " << energy << " kWh\n";
        std::cout << "Constraint Violations: " << violations << "\n";
        std::cout << "Sample Outputs (every 4 hours):\n";
        std::cout << "Time(h) | Temp(°C) | Hum(%) | Power(kW) | Humid(kg/h)\n";
        for (int k = 0; k < n_sim; k += 40) {
            printf("%7.1f | %8.2f | %6.2f | %9.2f | %11.3f\n",
                   k * mpc.model.Ts, T_traj[k], H_traj[k], P_traj[k], Q_traj[k]);
        }
    }

    return 0;
}
