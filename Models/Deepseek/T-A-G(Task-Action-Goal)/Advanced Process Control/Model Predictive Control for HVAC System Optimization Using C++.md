#include <iostream>
#include <vector>
#include <cmath>
#include <Eigen/Dense>
#include <Eigen/Sparse>
#include <qpOASES.hpp>

using namespace std;
using namespace Eigen;

// Parameters
const double Ts = 5; // Sampling time in minutes
const int N = 12;  // Prediction horizon (2 hours)
const double T_set = 22.0; // Desired temperature (°C)
const double H_set = 50.0; // Desired humidity (%)

// State vector: [temperature, humidity]
// Input vector: [heater power, humidifier power]

// Plant model parameters
const double A11 = 0.98; // Temperature decay factor
const double A12 = 0.01; // Humidity influence on temperature
const double A21 = 0.02; // Temperature influence on humidity
const double A22 = 0.97; // Humidity decay factor
const double B11 = 0.01; // Heater influence on temperature
const double B12 = 0.00; // Humidifier influence on temperature
const double B21 = 0.00; // Heater influence on humidity
const double B22 = 0.01; // Humidifier influence on humidity

// Constraints
const double U_min_heater = 0.0; // Minimum heater power
const double U_max_heater = 1.0; // Maximum heater power
const double U_min_humidifier = 0.0; // Minimum humidifier power
const double U_max_humidifier = 1.0; // Maximum humidifier power
const double T_min = 18.0; // Minimum allowed temperature
const double T_max = 26.0; // Maximum allowed temperature
const double H_min = 30.0; // Minimum allowed humidity
const double H_max = 70.0; // Maximum allowed humidity

// Cost function weights
const double Q_T = 1.0; // Weight for temperature deviation
const double Q_H = 1.0; // Weight for humidity deviation
const double R_U = 0.1; // Weight for control effort

// External conditions and occupancy levels
double external_temp = 15.0; // External temperature (°C)
double occupancy_level = 1.0; // Occupancy level (0-1)

// Plant model
void plant_model(Vector2d& x, Vector2d u, Vector2d& x_next) {
    x_next(0) = A11 * x(0) + A12 * x(1) + B11 * u(0) + B12 * u(1) + 0.1 * external_temp * occupancy_level;
    x_next(1) = A21 * x(0) + A22 * x(1) + B21 * u(0) + B22 * u(1) + 0.05 * external_temp * occupancy_level;
}

// MPC Controller
class MPCController {
public:
    MPCController(int N, double Ts, double Q_T, double Q_H, double R_U,
                  double U_min_heater, double U_max_heater, double U_min_humidifier, double U_max_humidifier,
                  double T_min, double T_max, double H_min, double H_max)
        : N(N), Ts(Ts), Q_T(Q_T), Q_H(Q_H), R_U(R_U),
          U_min_heater(U_min_heater), U_max_heater(U_max_heater),
          U_min_humidifier(U_min_humidifier), U_max_humidifier(U_max_humidifier),
          T_min(T_min), T_max(T_max), H_min(H_min), H_max(H_max) {}

    void solve(const Vector2d& x0, const VectorXd& demand, MatrixXd& u_opt) {
        int nVars = 2 * N;
        int nEqns = 2 * N;
        int nIneqs = 4 * N;

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
            H[k * 2][k * 2] += Q_T;
            H[k * 2 + 1][k * 2 + 1] += Q_H;
            g[k * 2] -= Q_T * demand(k * 2);
            g[k * 2 + 1] -= Q_H * demand(k * 2 + 1);
        }

        for (int k = 0; k < N - 1; ++k) {
            H[k * 2][k * 2 + 2] += R_U;
            H[k * 2 + 1][k * 2 + 3] += R_U;
        }

        // Equality constraints
        memset(A, 0, sizeof(real_t) * nEqns * nVars);
        memset(lbA, 0, sizeof(real_t) * nEqns);
        memset(ubA, 0, sizeof(real_t) * nEqns);

        Vector2d x_prev = x0;
        for (int k = 0; k < N; ++k) {
            A[k * 2][k * 2] = 1.0;
            A[k * 2 + 1][k * 2 + 1] = 1.0;

            if (k > 0) {
                A[k * 2][k * 2 - 2] = -A11;
                A[k * 2][k * 2 - 1] = -A12;
                A[k * 2][k * 2] = 1.0;
                A[k * 2][k * 2 + 2] = B11;
                A[k * 2][k * 2 + 3] = B12;

                A[k * 2 + 1][k * 2 - 2] = -A21;
                A[k * 2 + 1][k * 2 - 1] = -A22;
                A[k * 2 + 1][k * 2 + 1] = 1.0;
                A[k * 2 + 1][k * 2 + 2] = B21;
                A[k * 2 + 1][k * 2 + 3] = B22;
            } else {
                A[k * 2][k * 2] = 1.0;
                A[k * 2 + 1][k * 2 + 1] = 1.0;
            }
        }

        // Inequality constraints
        memset(lb, 0, sizeof(real_t) * nVars);
        memset(ub, 0, sizeof(real_t) * nVars);

        for (int k = 0; k < N; ++k) {
            lb[k * 2] = U_min_heater;
            ub[k * 2] = U_max_heater;
            lb[k * 2 + 1] = U_min_humidifier;
            ub[k * 2 + 1] = U_max_humidifier;
        }

        // Initial guess
        real_t* xOpt = new real_t[nVars];
        for (int k = 0; k < N; ++k) {
            xOpt[k * 2] = 0.5;
            xOpt[k * 2 + 1] = 0.5;
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

        u_opt.block(0, 0, 2, 1) = Map<Vector2d>(xOpt).transpose();

        delete[] xOpt;
    }

private:
    int N;
    double Ts;
    double Q_T;
    double Q_H;
    double R_U;
    double U_min_heater;
    double U_max_heater;
    double U_min_humidifier;
    double U_max_humidifier;
    double T_min;
    double T_max;
    double H_min;
    double H_max;
};

int main() {
    // Simulation parameters
    int simulation_steps = 120; // 2 hours of simulation at 5-minute intervals
    Vector2d x0(20.0, 40.0); // Initial state: [temperature, humidity]

    // Demand signal (desired temperature and humidity)
    VectorXd demand(2 * N);
    for (int k = 0; k < N; ++k) {
        demand(k * 2) = T_set;
        demand(k * 2 + 1) = H_set;
    }

    // Simulation results
    MatrixXd x(simulation_steps, 2);
    MatrixXd u(simulation_steps, 2);

    // MPC controller
    MPCController mpc(N, Ts, Q_T, Q_H, R_U, U_min_heater, U_max_heater, U_min_humidifier, U_max_humidifier, T_min, T_max, H_min, H_max);

    // Simulation loop
    Vector2d x_current = x0;
    for (int t = 0; t < simulation_steps; ++t) {
        // Solve MPC problem
        MatrixXd u_opt(N, 2);
        mpc.solve(x_current, demand, u_opt);

        // Apply first control action
        u.row(t) = u_opt.row(0);
        Vector2d u_action = u.row(t).transpose();

        // Simulate plant
        Vector2d x_next;
        plant_model(x_current, u_action, x_next);

        // Store results
        x.row(t) = x_current.transpose();
        x_current = x_next;

        // Update demand signal for next prediction horizon
        for (int k = 0; k < N - 1; ++k) {
            demand.segment(2 * k, 2) = demand.segment(2 * (k + 1), 2);
        }
        demand.segment(2 * (N - 1), 2) << T_set, H_set;

        // Update external conditions and occupancy levels
        external_temp = 15.0 + 5.0 * sin(0.1 * t); // Sinusoidal external temperature
        occupancy_level = 0.5 + 0.5 * sin(0.05 * t); // Sinusoidal occupancy level
    }

    // Output results
    cout << "Simulation Results:" << endl;
    cout << "Time\tTemperature\tHumidity\tHeater Power\tHumidifier Power" << endl;
    for (int t = 0; t < simulation_steps; ++t) {
        cout << t * Ts << "\t" << x(t, 0) << "\t\t" << x(t, 1) << "\t\t" << u(t, 0) << "\t\t" << u(t, 1) << endl;
    }

    return 0;
}



