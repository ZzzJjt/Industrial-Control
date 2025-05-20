#include <iostream>
#include <vector>
#include <Eigen/Dense>
#include <cmath>
#include <random>
#include <iomanip>

using namespace Eigen;
using namespace std;

// Step 1: Define Wind Farm Parameters
struct WindFarmParams {
    double J = 1e7;    // Turbine inertia (kg·m²)
    double eta = 0.4;  // Power conversion efficiency
    double rho = 1.225; // Air density (kg/m³)
    double A = 5000.0; // Rotor swept area (m²)
    double BESS_cap = 1e6; // Battery capacity (Wh)
    double P_nom = 2e6; // Nominal turbine power (W)
    double P_max = 2.5e6; // Max turbine power (W)
    double P_min = 0.0;   // Min turbine power (W)
    double w_max = 2.0;   // Max rotor speed (rad/s)
    double w_min = 0.5;   // Min rotor speed (rad/s)
    double SOC_max = 0.9;  // Max state of charge
    double SOC_min = 0.2;  // Min state of charge
    double P_bess_max = 5e5; // Max battery power (W)
    double dt = 10.0;     // Time step (s)
    double T_sim = 3600.0; // Simulation duration (s)
};

// Step 2: Wind Turbine and BESS Dynamic Model
class WindFarmSystem {
private:
    WindFarmParams params;
    MatrixXd A, B, C; // State-space matrices
    VectorXd x;       // State: [w, SOC] (rotor speed, battery SOC)
    
public:
    WindFarmSystem(const WindFarmParams& p) : params(p) {
        // Linearized state-space model
        A = MatrixXd(2, 2);
        B = MatrixXd(2, 2);
        C = MatrixXd(2, 2);
        A << 1.0, 0.0, 0.0, 1.0; // Simplified dynamics (discrete-time)
        B << params.dt / params.J, 0.0,
             0.0, -params.dt / (3600.0 * params.BESS_cap);
        C.setIdentity();
        x = VectorXd(2);
        x << params.w_min + 0.5, 0.5; // Initial rotor speed and SOC
    }
    
    VectorXd update(const VectorXd& u, double v_wind) {
        // u: [P_turb, P_bess] (turbine power, battery power)
        // v_wind: wind speed (m/s)
        double P_wind = 0.5 * params.rho * params.A * pow(v_wind, 3) * params.eta;
        double torque = P_wind / x(0); // Torque from wind
        x(0) += params.dt * (torque - u(0) / x(0)) / params.J; // Rotor speed
        x(1) += -params.dt * u(1) / (3600.0 * params.BESS_cap); // SOC
        x(0) = std::max(params.w_min, std::min(params.w_max, x(0)));
        x(1) = std::max(params.SOC_min, std::min(params.SOC_max, x(1)));
        return C * x;
    }
    
    VectorXd getState() const { return x; }
    double getPowerOutput(const VectorXd& u) const {
        return u(0) + u(1); // Total power (turbine + battery)
    }
};

// Step 3: MPC Controller
class MPCController {
private:
    WindFarmParams params;
    int Np = 10; // Prediction horizon
    int Nc = 3;  // Control horizon
    MatrixXd Q, R; // Weight matrices
    vector<double> load_forecast; // Grid load demand
    vector<double> wind_forecast; // Wind speed forecast
    
public:
    MPCController(const WindFarmParams& p) : params(p) {
        Q = MatrixXd(2, 2);
        Q << 1.0, 0.0, 0.0, 10.0; // Weight rotor speed, SOC
        R = MatrixXd(2, 2);
        R << 0.01, 0.0, 0.0, 0.01; // Weight control effort
        generateForecasts();
    }
    
    void generateForecasts() {
        random_device rd;
        mt19937 gen(rd());
        normal_distribution<> d(0.0, 0.5);
        int N = static_cast<int>(params.T_sim / params.dt);
        load_forecast.resize(N);
        wind_forecast.resize(N);
        for (int k = 0; k < N; ++k) {
            double t = k * params.dt / 3600.0; // Time in hours
            load_forecast[k] = params.P_nom * (0.8 + 0.2 * sin(2 * M_PI * t)) + d(gen) * 1e5;
            load_forecast[k] = std::max(params.P_min, std::min(params.P_max, load_forecast[k]));
            wind_forecast[k] = 10.0 + 3.0 * sin(2 * M_PI * t / 2.0) + d(gen);
            wind_forecast[k] = std::max(5.0, std::min(15.0, wind_forecast[k]));
        }
    }
    
    VectorXd computeControl(const VectorXd& x, int k) {
        VectorXd u_opt(2 * Nc);
        u_opt.setZero();
        VectorXd u_prev = VectorXd::Zero(2);
        
        auto objective = [&](const VectorXd& u) {
            double cost = 0.0;
            VectorXd x_pred = x;
            for (int i = 0; i < Np; ++i) {
                int u_idx = std::min(i, Nc - 1) * 2;
                VectorXd u_i(2);
                u_i << u(u_idx), u(u_idx + 1);
                double v_wind = k + i < wind_forecast.size() ? wind_forecast[k + i] : wind_forecast.back();
                double P_wind = 0.5 * params.rho * params.A * pow(v_wind, 3) * params.eta;
                double torque = P_wind / x_pred(0);
                x_pred(0) += params.dt * (torque - u_i(0) / x_pred(0)) / params.J;
                x_pred(1) += -params.dt * u_i(1) / (3600.0 * params.BESS_cap);
                x_pred(0) = std::max(params.w_min, std::min(params.w_max, x_pred(0)));
                x_pred(1) = std::max(params.SOC_min, std::min(params.SOC_max, x_pred(1)));
                VectorXd e = x_pred - VectorXd(2 << 1.5, 0.5); // Reference state
                double P_total = u_i(0) + u_i(1);
                double P_load = k + i < load_forecast.size() ? load_forecast[k + i] : load_forecast.back();
                double P_error = (P_total - P_load) / params.P_nom;
                cost += e.transpose() * Q * e + P_error * P_error;
                if (i < Nc) {
                    VectorXd du = u_i - (i == 0 ? u_prev : VectorXd(2 << u(u_idx - 2), u(u_idx - 1)));
                    cost += u_i.transpose() * R * u_i + du.transpose() * R * du;
                }
            }
            return cost;
        };
        
        vector<double> lb(2 * Nc), ub(2 * Nc);
        for (int i = 0; i < Nc; ++i) {
            lb[2*i] = params.P_min;
            ub[2*i] = params.P_max;
            lb[2*i + 1] = -params.P_bess_max;
            ub[2*i + 1] = params.P_bess_max;
        }
        
        // Simplified gradient-based optimization
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
    
    const vector<double>& getLoadForecast() const { return load_forecast; }
    const vector<double>& getWindForecast() const { return wind_forecast; }
};

// Step 4: Simulation
int main() {
    WindFarmParams params;
    WindFarmSystem system(params);
    MPCController mpc(params);
    
    int N = static_cast<int>(params.T_sim / params.dt);
    vector<double> w_history(N), SOC_history(N), P_turb_history(N), P_bess_history(N), P_total_history(N);
    VectorXd x = system.getState();
    w_history[0] = x(0);
    SOC_history[0] = x(1);
    
    // Simulate two scenarios: normal and high wind variability
    for (int scenario = 0; scenario < 2; ++scenario) {
        cout << "\nScenario " << (scenario == 0 ? "Normal" : "High") << " Wind Variability\n";
        cout << "Time (min) | Rotor Speed (rad/s) | SOC | Turbine Power (MW) | Battery Power (MW) | Total Power (MW) | Load (MW)\n";
        cout << string(90, '-') << endl;
        
        system = WindFarmSystem(params);
        x = system.getState();
        w_history[0] = x(0);
        SOC_history[0] = x(1);
        P_turb_history[0] = 0.0;
        P_bess_history[0] = 0.0;
        P_total_history[0] = 0.0;
        
        double total_energy = 0.0;
        for (int k = 0; k < N; ++k) {
            double v_wind = mpc.getWindForecast()[k];
            if (scenario == 1) {
                v_wind *= 1.3; // 30% higher variability
            }
            
            VectorXd u = mpc.computeControl(x, k);
            P_turb_history[k] = u(0);
            P_bess_history[k] = u(1);
            P_total_history[k] = system.getPowerOutput(u);
            
            x = system.update(u, v_wind);
            w_history[k] = x(0);
            SOC_history[k] = x(1);
            
            total_energy += P_total_history[k] * params.dt / 3600.0; // Wh
            
            if (k % static_cast<int>(600.0 / params.dt) == 0) {
                cout << fixed << setprecision(2);
                cout << k * params.dt / 60.0 << "\t| " << w_history[k] << "\t| "
                     << SOC_history[k] << "\t| " << P_turb_history[k] / 1e6 << "\t| "
                     << P_bess_history[k] / 1e6 << "\t| " << P_total_history[k] / 1e6 << "\t| "
                     << mpc.getLoadForecast()[k] / 1e6 << endl;
            }
        }
        
        // Performance Metrics
        double mse_P = 0.0;
        int settling_idx = N - 1;
        for (int k = 0; k < N; ++k) {
            double e = P_total_history[k] - mpc.getLoadForecast()[k];
            mse_P += e * e;
            if (k > 100 && abs(e) < 0.05 * params.P_nom) {
                settling_idx = std::min(settling_idx, k);
            }
        }
        mse_P /= N * 1e12; // Normalize to MW^2
        double settling_time = settling_idx * params.dt / 60.0;
        double efficiency = total_energy / (params.T_sim / 3600.0) / params.P_nom * 100.0;
        
        cout << "\nPerformance Metrics:\n";
        cout << "MSE Power: " << mse_P << " MW^2\n";
        cout << "Settling Time: " << settling_time << " min\n";
        cout << "Energy Efficiency: " << efficiency << "%\n";
        cout << "Total Energy: " << total_energy / 1e6 << " MWh\n";
    }
    
    return 0;
}
