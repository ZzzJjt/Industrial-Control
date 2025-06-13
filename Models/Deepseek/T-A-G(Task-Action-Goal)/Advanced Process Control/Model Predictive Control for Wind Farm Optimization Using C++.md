#include <iostream>
#include <vector>
#include <cmath>
#include <Eigen/Dense>
#include <qpOASES.hpp>

using namespace std;
using namespace Eigen;

// Parameters
const double Ts = 1.0; // Sampling time in seconds
const int N = 20;      // Prediction horizon
const double P_max = 5000.0; // Maximum power output per turbine (Watts)
const double E_storage_max = 10000.0; // Maximum energy storage capacity (Joules)

// Wind farm parameters
const int num_turbines = 5;
const double Cp = 0.44; // Power coefficient
const double rho = 1.225; // Air density (kg/m^3)
const double A = 100.0; // Swept area of each turbine (m^2)

// Grid connection parameters
const double P_grid_target = 20000.0; // Target grid power output (Watts)

// Cost function weights
const double Q_P = 1.0; // Weight for power deviation from target
const double Q_E = 0.1; // Weight for energy storage deviation
const double R_U = 0.01; // Weight for control effort

// Constraints
const double U_min = 0.0; // Minimum control input (blade pitch angle adjustment)
const double U_max = 1.0; // Maximum control input (blade pitch angle adjustment)
const double P_min = 0.0; // Minimum power output per turbine
const double P_max = 5000.0; // Maximum power output per turbine
const double E_storage_min = 0.0; // Minimum energy storage
const double E_storage_max = 10000.0; // Maximum energy storage

// Wind speed model (sinusoidal for simulation purposes)
double wind_speed(double t) {
    return 10.0 + 5.0 * sin(0.1 * t); // Mean wind speed + fluctuation
}

// Turbine power output model
double turbine_power(double wind_v, double blade_pitch_angle) {
    if (wind_v < 3.0 || wind_v > 25.0) {
        return 0.0; // Below cut-in or above cut-out wind speed
    }
    double V_rated = 12.0; // Rated wind speed
    double P_rated = P_max; // Rated power output
    return 0.5 * rho * A * pow(wind_v, 3) * Cp * blade_pitch_angle / pow(V_rated, 3) * (P_rated / Cp);
}

// Plant model
void plant_model(VectorXd& x, VectorXd u, VectorXd& x_next, double t) {
    VectorXd P_turbines(num_turbines);
    double total_power = 0.0;
    for (int i = 0; i < num_turbines; ++i) {
        double wind_v = wind_speed(t);
        P_turbines(i) = turbine_power(wind_v, u(i));
        total_power += P_turbines(i);
    }
    
    double E_storage = x(0);
    double delta_E_storage = (total_power - P_grid_target) * Ts;
    E_storage += delta_E_storage;
    E_storage = max(E_storage_min, min(E_storage, E_storage_max)); // Clamp within bounds
    
    x_next(0) = E_storage;
    x_next.segment(1, num_turbines) = P_turbines;
}

// MPC Controller
class MPCController {
public:
    MPCController(int N, double Ts, double Q_P, double Q_E, double R_U,
                  double U_min, double U_max, double P_min, double P_max,
                  double E_storage_min, double E_storage_max, double P_grid_target)
        : N(N), Ts(Ts), Q_P(Q_P), Q_E(Q_E), R_U(R_U),
          U_min(U_min), U_max(U_max),
          P_min(P_min), P_max(P_max),
          E_storage_min(E_storage_min), E_storage_max(E_storage_max),
          P_grid_target(P_grid_target) {}

    void solve(const VectorXd& x0, MatrixXd& u_opt) {
        int nVars = N * num_turbines;
        int nEqns = N;
        int nIneqs = 2 * N * num_turbines + 2;

        SQProblem example(nVars, nEqns);
        real_t* H = new real_t[nVars * nVars];
        real_t* g = new real_t[nVars];
        real_t* A = new real_t[nEqns * nVars];
        real_t* lb = new real_t[nVars];
        real_t* ub = new real_t[nVars];
        real_t* lbA = new real_t[nEqns];
        real_t* ubA = new real_t[nEqns];

        // Objective function
        memset(H, 0, sizeof(real_t) * nVars * nVars);
        memset(g, 0, sizeof(real_t) * nVars);

        for (int k = 0; k < N; ++k) {
            for (int i = 0; i < num_turbines; ++i) {
                int idx = k * num_turbines + i;
                H[idx][idx] += Q_P + R_U;
                g[idx] -= Q_P * P_grid_target;
            }
        }

        // Equality constraints
        memset(A, 0, sizeof(real_t) * nEqns * nVars);
        memset(lbA, 0, sizeof(real_t) * nEqns);
        memset(ubA, 0, sizeof(real_t) * nEqns);

        VectorXd x_prev = x0;
        for (int k = 0; k < N; ++k) {
            for (int i = 0; i < num_turbines; ++i) {
                int idx = k * num_turbines + i;
                A[k][idx] = 1.0;
            }
        }

        // Inequality constraints
        memset(lb, 0, sizeof(real_t) * nVars);
        memset(ub, 0, sizeof(real_t) * nVars);

        for (int k = 0; k < N; ++k) {
            for (int i = 0; i < num_turbines; ++i) {
                int idx = k * num_turbines + i;
                lb[idx] = U_min;
                ub[idx] = U_max;
            }
        }

        // Initial guess
        real_t* xOpt = new real_t[nVars];
        for (int k = 0; k < N; ++k) {
            for (int i = 0; i < num_turbines; ++i) {
                int idx = k * num_turbines + i;
                xOpt[idx] = 0.5;
            }
        }

        Options options;
        options.setToDefault();
        options.printLevel = PL_NONE;
        example.setOptions(options);

        int nWSR = 100;
        ReturnValue returnvalue = example.init(H, g, A, lb, ub, lbA, ubA, nWSR, xOpt);

        if (returnvalue != SUCCESSFUL_RETURN) {
            cerr << "Initialization failed!" << endl;
        }

        delete[] H;
        delete[] g;
        delete[] A;
        delete[] lb;
        delete[] ub;
        delete[] lbA;
        delete[] ubA;

        Map<MatrixXd>(xOpt, N, num_turbines).transpose().evalTo(u_opt);

        delete[] xOpt;
    }

private:
    int N;
    double Ts;
    double Q_P;
    double Q_E;
    double R_U;
    double U_min;
    double U_max;
    double P_min;
    double P_max;
    double E_storage_min;
    double E_storage_max;
    double P_grid_target;
};

int main() {
    // Simulation parameters
    int simulation_steps = 100; // Total simulation time in seconds
    VectorXd x0(1 + num_turbines); // Initial state: [energy_storage, P_turbine_1, ..., P_turbine_n]
    x0.setZero();

    // Simulation results
    MatrixXd x(simulation_steps, 1 + num_turbines);
    MatrixXd u(simulation_steps, num_turbines);

    // MPC controller
    MPCController mpc(N, Ts, Q_P, Q_E, R_U, U_min, U_max, P_min, P_max, E_storage_min, E_storage_max, P_grid_target);

    // Simulation loop
    VectorXd x_current = x0;
    for (int t = 0; t < simulation_steps; ++t) {
        // Solve MPC problem
        MatrixXd u_opt(N, num_turbines);
        mpc.solve(x_current, u_opt);

        // Apply first control action
        u.row(t) = u_opt.row(0);
        VectorXd u_action = u.row(t).transpose();

        // Simulate plant
        VectorXd x_next(1 + num_turbines);
        plant_model(x_current, u_action, x_next, t * Ts);

        // Store results
        x.row(t) = x_current.transpose();
        x_current = x_next;
    }

    // Output results
    cout << "Simulation Results:" << endl;
    cout << "Time\tEnergy Storage\tPower Output (Turbines)\tControl Inputs" << endl;
    for (int t = 0; t < simulation_steps; ++t) {
        cout << t * Ts << "\t" << x(t, 0) << "\t\t";
        for (int i = 0; i < num_turbines; ++i) {
            cout << x(t, i + 1) << " ";
        }
        cout << "\t\t";
        for (int i = 0; i < num_turbines; ++i) {
            cout << u(t, i) << " ";
        }
        cout << endl;
    }

    return 0;
}



