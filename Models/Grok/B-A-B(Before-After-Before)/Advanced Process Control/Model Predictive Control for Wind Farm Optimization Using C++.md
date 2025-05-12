#include <iostream>
#include <vector>
#include <cmath>
#include <random>
#include <Eigen/Dense>
#include <OsqpEigen/OsqpEigen.h>
#include <fstream>

using namespace Eigen;
using namespace std;

// Wind Farm Parameters
const double dt = 0.1; // Time step (hours)
const int N_sim = 200; // Simulation steps (20 hours)
const double P_rated = 2.0; // Rated turbine power (MW)
const double v_rated = 12.0; // Rated wind speed (m/s)
const double v_cut_in = 3.0; // Cut-in wind speed (m/s)
const double v_cut_out = 25.0; // Cut-out wind speed (m/s)
const double E_max = 10.0; // Battery capacity (MWh)
const double E_min = 1.0; // Min battery energy (MWh)
const double P_charge_max = 1.0; // Max charge/discharge rate (MW)
const double eta_charge = 0.95; // Charging efficiency
const double eta_discharge = 0.95; // Discharging efficiency
const double P_grid_max = 2.5; // Max grid dispatch (MW)
const double P_grid_min = 0.0; // Min grid dispatch (MW)

// MPC Parameters
const int Np = 10; // Prediction horizon
const int Nc = 3; // Control horizon
const double Q_weight = 10.0; // Power tracking weight
const double R_weight = 0.1; // Control effort weight
const double S_weight = 1.0; // Battery usage weight
const double P_ref = 1.8; // Reference grid power (MW)

// Wind Turbine Model
class WindTurbine {
public:
    double P; // Power output (MW)
    
    WindTurbine() : P(0.0) {}
    
    double update(double v, double P_cmd) {
        // Power curve approximation
        double P_max = 0.0;
        if (v >= v_cut_in && v <= v_rated) {
            P_max = P_rated * pow(v - v_cut_in, 3) / pow(v_rated - v_cut_in, 3);
        } else if (v > v_rated && v <= v_cut_out) {
            P_max = P_rated;
        }
        P = std::max(0.0, std::min(P_max, P_cmd));
        return P;
    }
};

// Battery Storage Model
class Battery {
public:
    double E; // Stored energy (MWh)
    
    Battery(double E0) : E(E0) {}
    
    void update(double P_batt) {
        if (P_batt >= 0) { // Charging
            E += P_batt * dt * eta_charge;
        } else { // Discharging
            E += P_batt * dt / eta_discharge;
        }
        E = std::max(E_min, std::min(E_max, E));
    }
};

// Wind Speed Model
class Environment {
private:
    std::mt19937 gen;
    std::normal_distribution<double> noise;
    
public:
    Environment() : gen(42), noise(0.0, 0.5) {}
    
    double get_wind_speed(double t) {
        // Wind speed: mean + sinusoidal variation + noise
        double v = 10.0 + 3.0 * sin(2 * M_PI * t / 24.0) + noise(gen);
        return std::max(v_cut_in, std::min(v_cut_out, v));
    }
};

// MPC Controller
class MPCController {
private:
    int Np, Nc;
    double dt;
    MatrixXd Q, R, S;
    double P_ref;
    OsqpEigen::Solver solver;
    
public:
    MPCController(int np, int nc, double dt_, double P_ref_)
        : Np(np), Nc(nc), dt(dt_), P_ref(P_ref_) {
        Q = Q_weight * MatrixXd::Identity(Np, Np); // Power tracking
        R = R_weight * MatrixXd::Identity(2 * Nc, 2 * Nc); // Control effort
        S = S_weight * MatrixXd::Identity(Np, Np); // Battery energy deviation
        setup_solver();
    }
    
    void setup_solver() {
        int n = 2 * Nc; // Decision variables: [P_cmd, P_batt]
        int m = 4 * Nc + 2 * Np; // Constraints: input bounds + battery bounds
        
        // Hessian matrix
        MatrixXd H = 2.0 * R;
        SparseMatrix<double> H_sparse = H.sparseView();
        
        // Linear term
        VectorXd f = VectorXd::Zero(n);
        
        // Constraint matrices
        MatrixXd A = MatrixXd::Zero(m, n);
        VectorXd lb = VectorXd::Zero(m);
        VectorXd ub = VectorXd::Zero(m);
        
        // Input constraints
        for (int i = 0; i < Nc; ++i) {
            A(i, 2 * i) = 1.0; // P_cmd
            A(Nc + i, 2 * i + 1) = 1.0; // P_batt
            A(2 * Nc + i, 2 * i) = -1.0;
            A(3 * Nc + i, 2 * i + 1) = -1.0;
            lb(i) = 0.0;
            ub(i) = P_rated;
            lb(Nc + i) = -P_charge_max;
            ub(Nc + i) = P_charge_max;
            lb(2 * Nc + i) = -P_rated;
            ub(2 * Nc + i) = 0.0;
            lb(3 * Nc + i) = -P_charge_max;
            ub(3 * Nc + i) = P_charge_max;
        }
        
        // Battery energy constraints
        for (int i = 0; i < Np; ++i) {
            A(4 * Nc + i, 2 * (Nc - 1) + 1) = dt * (i < Nc ? eta_charge : eta_discharge);
            lb(4 * Nc + i) = E_min;
            ub(4 * Nc + i) = E_max;
        }
        
        SparseMatrix<double> A_sparse = A.sparseView();
        
        // Initialize solver
        solver.settings()->setVerbosity(false);
        solver.data()->setNumberOfVariables(n);
        solver.data()->setNumberOfConstraints(m);
        solver.data()->setHessianMatrix(H_sparse);
        solver.data()->setGradient(f);
        solver.data()->setLinearConstraintsMatrix(A_sparse);
        solver.data()->setLowerBound(lb);
        solver.data()->setUpperBound(ub);
        solver.initSolver();
    }
    
    Vector2d control(double E, double v, double t, const WindTurbine& turbine, const Battery& battery) {
        VectorXd f = VectorXd::Zero(2 * Nc);
        double P_turb = turbine.P;
        double E_batt = E;
        
        // Predict wind speeds
        vector<double> v_pred(Np);
        for (int i = 0; i < Np; ++i) {
            v_pred[i] = get_wind_speed(t + i * dt);
        }
        
        // Compute gradient
        for (int i = 0; i < Np; ++i) {
            double P_max = 0.0;
            if (v_pred[i] >= v_cut_in && v_pred[i] <= v_rated) {
                P_max = P_rated * pow(v_pred[i] - v_cut_in, 3) / pow(v_rated - v_cut_in, 3);
            } else if (v_pred[i] > v_rated && v_pred[i] <= v_cut_out) {
                P_max = P_rated;
            }
            double P_cmd = i < Nc ? P_turb : P_turb;
            double P_batt = i < Nc ? 0.0 : 0.0;
            double P_grid = std::min(P_cmd + P_batt, P_grid_max);
            f.segment(0, 2 * Nc) += Q(i, i) * (P_grid - P_ref);
            
            // Update battery
            if (P_batt >= 0) {
                E_batt += P_batt * dt * eta_charge;
            } else {
                E_batt += P_batt * dt / eta_discharge;
            }
            E_batt = std::max(E_min, std::min(E_max, E_batt));
            f.segment(0, 2 * Nc) += S(i, i) * (E_batt - 0.5 * E_max);
        }
        f *= 2.0;
        
        // Solve optimization
        solver.updateGradient(f);
        solver.solve();
        
        VectorXd u_opt = solver.getSolution();
        return Vector2d(u_opt(0), u_opt(1));
    }
    
    double get_wind_speed(double t) {
        Environment env;
        return env.get_wind_speed(t);
    }
};

// Main simulation
int main() {
    // Initialize system
    WindTurbine turbine;
    Battery battery(0.5 * E_max);
    Environment env;
    MPCController mpc(Np, Nc, dt, P_ref);
    
    // Storage
    vector<double> P_turb(N_sim), P_grid(N_sim), E_batt(N_sim), v_wind(N_sim);
    vector<double> P_cmd(N_sim), P_batt(N_sim);
    P_turb[0] = turbine.P;
    E_batt[0] = battery.E;
    v_wind[0] = env.get_wind_speed(0);
    P_grid[0] = 0.0;
    P_cmd[0] = 0.0;
    P_batt[0] = 0.0;
    
    // Simulation loop
    for (int i = 1; i < N_sim; ++i) {
        double t = i * dt;
        v_wind[i] = env.get_wind_speed(t);
        
        // Compute control action
        Vector2d u = mpc.control(battery.E, v_wind[i], t, turbine, battery);
        P_cmd[i] = u(0);
        P_batt[i] = u(1);
        
        // Update system
        P_turb[i] = turbine.update(v_wind[i], P_cmd[i]);
        battery.update(P_batt[i]);
        P_grid[i] = std::min(P_turb[i] + P_batt[i], P_grid_max);
        E_batt[i] = battery.E;
    }
    
    // Performance metrics
    double P_error = 0.0;
    double energy = 0.0;
    for (int i = 0; i < N_sim; ++i) {
        P_error += std::abs(P_grid[i] - P_ref);
        energy += P_grid[i] * dt;
    }
    P_error /= N_sim;
    
    // Output results
    cout << "Mean Power Tracking Error: " << P_error << " MW" << endl;
    cout << "Total Energy Delivered: " << energy << " MWh" << endl;
    cout << "Final Battery Energy: " << E_batt[N_sim-1] << " MWh" << endl;
    cout << "Constraints Satisfied: "
         << (std::all_of(E_batt.begin(), E_batt.end(),
                        [](double e) { return e >= E_min && e <= E_max; }) &&
             std::all_of(P_grid.begin(), P_grid.end(),
                        [](double p) { return p >= P_grid_min && p <= P_grid_max; }) &&
             std::all_of(P_cmd.begin(), P_cmd.end(),
                        [](double p) { return p >= 0.0 && p <= P_rated; }) &&
             std::all_of(P_batt.begin(), P_batt.end(),
                        [](double p) { return p >= -P_charge_max && p <= P_charge_max; }))
         << endl;
    
    // Save data for visualization
    ofstream out("windfarm_data.txt");
    for (int i = 0; i < N_sim; ++i) {
        out << i * dt << " " << P_turb[i] << " " << P_grid[i] << " "
            << E_batt[i] << " " << v_wind[i] << endl;
    }
    out.close();
    
    return 0;
}
