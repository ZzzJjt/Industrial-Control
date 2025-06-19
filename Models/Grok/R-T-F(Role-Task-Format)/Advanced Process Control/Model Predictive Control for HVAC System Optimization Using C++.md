#include <iostream>
#include <vector>
#include <Eigen/Dense>
#include <cmath>
#include <random>
#include <iomanip>

using namespace Eigen;
using namespace std;

// Step 1: Define HVAC System Parameters and Dynamics
struct HVACParams {
    double C_t = 1e6; // Thermal capacitance (J/K)
    double R_t = 0.1; // Thermal resistance (K/W)
    double C_h = 1e4; // Humidity capacitance (kg/m^3)
    double R_h = 0.2; // Humidity resistance (m^3/kg)
    double eta_c = 0.9; // Cooling efficiency
    double eta_h = 0.8; // Humidification efficiency
    double T_sp = 22.0; // Temperature setpoint (C)
    double H_sp = 0.01; // Humidity setpoint (kg/m^3)
    double P_max = 10000.0; // Max power (W)
    double T_min = 18.0, T_max = 26.0; // Temperature bounds (C)
    double H_min = 0.008, H_max = 0.012; // Humidity bounds (kg/m^3)
    double dt = 60.0; // Time step (s)
    double T_sim = 3600.0 * 24; // Simulation time (s)
};

// HVAC system dynamics (linearized state-space model)
class HVACSystem {
private:
    HVACParams params;
    MatrixXd A, B, C; // State-space matrices
    VectorXd x; // State: [T, H] (temperature, humidity)
    
public:
    HVACSystem(const HVACParams& p) : params(p) {
        // Initialize state-space model
        A = MatrixXd(2, 2);
        B = MatrixXd(2, 2);
        C = MatrixXd(2, 2);
        A << 1.0 - params.dt/(params.C_t * params.R_t), 0.0,
             0.0, 1.0 - params.dt/(params.C_h * params.R_h);
        B << -params.dt * params.eta_c / params.C_t, 0.0,
             0.0, params.dt * params.eta_h / params.C_h;
        C.setIdentity();
        x = VectorXd(2);
        x << params.T_sp, params.H_sp; // Initial state
    }
    
    VectorXd update(const VectorXd& u, const VectorXd& d) {
        // u: [P_cool, P_humid] (cooling, humidification power)
        // d: [T_out, H_out] (outside temperature, humidity)
        x = A * x + B * u + (VectorXd(2) << params.dt * d(0)/(params.C_t * params.R_t),
                           params.dt * d(1)/(params.C_h * params.R_h)).finished();
        x(0) = std::max(params.T_min, std::min(params.T_max, x(0))); // Constrain temperature
        x(1) = std::max(params.H_min, std::min(params.H_max, x(1))); // Constrain humidity
        return C * x;
    }
    
    VectorXd getState() const { return x; }
};

// Step 2: MPC Controller
class MPCController {
private:
    HVACParams params;
    int Np = 10; // Prediction horizon
    int Nc = 3;  // Control horizon
    MatrixXd Q, R; // Weight matrices
    vector<VectorXd> demand_forecast; // Occupancy-based demand
    
public:
    MPCController(const HVACParams& p) : params(p) {
        Q = MatrixXd(2, 2);
        Q << 10.0, 0.0, 0.0, 100.0; // Weight T, H errors
        R = MatrixXd(2, 2);
        R << 0.01, 0.0, 0.0, 0.01; // Weight control effort
        generateDemandForecast();
    }
    
    void generateDemandForecast() {
        // Simulate occupancy and weather disturbances
        random_device rd;
        mt19937 gen(rd());
        normal_distribution<> d(0.0, 0.5);
        int N = static_cast<int>(params.T_sim / params.dt);
        demand_forecast.resize(N, VectorXd(2));
        for (int k = 0; k < N; ++k) {
            double t = k * params.dt / 3600.0; // Time in hours
            double T_out = 30.0 + 5.0 * sin(2 * M_PI * t / 24.0) + d(gen); // Outside temp (C)
            double H_out = 0.015 + 0.005 * sin(2 * M_PI * t / 24.0) + d(gen) * 0.001; // Outside humidity
            double occ = (t >= 8 && t <= 18) ? 1.0 : 0.2; // Occupancy (high 8 AM - 6 PM)
            demand_forecast[k] << T_out * occ, H_out * occ;
        }
    }
    
    VectorXd computeControl(const VectorXd& x, int k) {
        // Simple MPC: optimize control sequence
        VectorXd u_opt(2 * Nc);
        u_opt.setZero(); // Initial guess
        VectorXd u_prev = VectorXd::Zero(2); // Previous control
        
        // Objective function
        auto objective = [&](const VectorXd& u) {
            double cost = 0.0;
            VectorXd x_pred = x;
            for (int i = 0; i < Np; ++i) {
                int u_idx = std::min(i, Nc - 1) * 2;
                VectorXd u_i(2);
                u_i << u(u_idx), u(u_idx + 1);
                VectorXd d = k + i < demand_forecast.size() ? demand_forecast[k + i] : demand_forecast.back();
                x_pred = (A * x_pred + B * u_i + (VectorXd(2) << params.dt * d(0)/(params.C_t * params.R_t),
                                  params.dt * d(1)/(params.C_h * params.R_h)).finished()).eval();
                VectorXd e = x_pred - VectorXd(2 << params.T_sp, params.H_sp);
                cost += e.transpose() * Q * e;
                if (i < Nc) {
                    VectorXd du = u_i - (i == 0 ? u_prev : VectorXd(2 << u(u_idx - 2), u(u_idx - 1)));
                    cost += u_i.transpose() * R * u_i + du.transpose() * R * du;
                }
            }
            return cost;
        };
        
        // Constraints
        vector<double> lb(2 * Nc, 0.0);
        vector<double> ub(2 * Nc, params.P_max);
        
        // Simple optimization (gradient-based, for demo purposes)
        // In practice, use a QP solver like QuadProg++
        double tol = 1e-4;
        double step = 0.1;
        VectorXd grad(2 * Nc);
        for (int iter = 0; iter < 100; ++iter) {
            double f = objective(u_opt);
            for (int j = 0; j < 2 * Nc; ++j) {
                VectorXd u_test = u_opt;
                u_test(j) += tol;
                grad(j) = (objective(u_test) - f) / tol;
            }
            u_opt -= step * grad;
            for (int j = 0; j < 2 * Nc; ++j) {
                u_opt(j) = std::max(lb[j], std::min(ub[j], u_opt(j)));
            }
            if (grad.norm() < 1e-3) break;
        }
        
        u_prev << u_opt(0), u_opt(1);
        return u_prev;
    }
};

// Step 3: Simulation
int main() {
    HVACParams params;
    HVACSystem system(params);
    MPCController mpc(params);
    
    int N = static_cast<int>(params.T_sim / params.dt);
    vector<double> T_history(N, params.T_sp);
    vector<double> H_history(N, params.H_sp);
    vector<double> P_cool_history(N, 0.0);
    vector<double> P_humid_history(N, 0.0);
    vector<double> energy_history(N, 0.0);
    
    // Simulate two scenarios: normal and high occupancy
    for (int scenario = 0; scenario < 2; ++scenario) {
        cout << "\nScenario " << (scenario == 0 ? "Normal" : "High") << " Occupancy\n";
        cout << "Time (h) | Temp (C) | Humidity (kg/m^3) | Cooling (W) | Humid (W) | Energy (kWh)\n";
        cout << string(70, '-') << endl;
        
        // Reset system
        system = HVACSystem(params);
        VectorXd x = system.getState();
        T_history[0] = x(0);
        H_history[0] = x(1);
        
        double total_energy = 0.0;
        for (int k = 0; k < N; ++k) {
            // Adjust demand for high occupancy scenario
            VectorXd d = mpc.generateDemandForecast()[k];
            if (scenario == 1 && k * params.dt / 3600.0 >= 8 && k * params.dt / 3600.0 <= 18) {
                d *= 1.5; // 50% higher demand
            }
            
            // Compute control
            VectorXd u = mpc.computeControl(x, k);
            P_cool_history[k] = u(0);
            P_humid_history[k] = u(1);
            
            // Update system
            x = system.update(u, d);
            T_history[k] = x(0);
            H_history[k] = x(1);
            
            // Energy consumption
            double energy = (u(0) + u(1)) * params.dt / 3600.0 / 1000.0; // kWh
            total_energy += energy;
            energy_history[k] = total_energy;
            
            // Output every hour
            if (k % static_cast<int>(3600.0 / params.dt) == 0) {
                cout << fixed << setprecision(1);
                cout << k * params.dt / 3600.0 << "\t| " << T_history[k] << "\t| "
                     << setprecision(4) << H_history[k] << "\t| " << setprecision(1)
                     << P_cool_history[k] << "\t| " << P_humid_history[k] << "\t| "
                     << energy_history[k] << endl;
            }
        }
        
        // Performance Metrics
        double mse_T = 0.0, mse_H = 0.0;
        int settling_idx = N - 1;
        for (int k = 0; k < N; ++k) {
            mse_T += pow(T_history[k] - params.T_sp, 2);
            mse_H += pow(H_history[k] - params.H_sp, 2);
            if (k > 100 && abs(T_history[k] - params.T_sp) < 0.5 &&
                abs(H_history[k] - params.H_sp) < 0.0005) {
                settling_idx = std::min(settling_idx, k);
            }
        }
        mse_T /= N;
        mse_H /= N;
        double settling_time = settling_idx * params.dt / 3600.0;
        
        cout << "\nPerformance Metrics:\n";
        cout << "MSE Temperature: " << mse_T << " C^2\n";
        cout << "MSE Humidity: " << mse_H << " (kg/m^3)^2\n";
        cout << "Settling Time: " << settling_time << " h\n";
        cout << "Total Energy: " << total_energy << " kWh\n";
    }
    
    return 0;
}
