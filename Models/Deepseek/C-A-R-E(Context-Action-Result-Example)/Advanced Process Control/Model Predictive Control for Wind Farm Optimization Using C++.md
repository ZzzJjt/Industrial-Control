#include <iostream>
#include <vector>
#include <cmath>
#include <Eigen/Dense>
#include <Eigen/Sparse>
#include <Eigen/QR>

using namespace std;
using namespace Eigen;

// Constants
const double dt = 1.0; // Time step (seconds)
const int N = 20; // Prediction horizon (time steps)
const int num_turbines = 5; // Number of turbines
const int num_batteries = 1; // Number of batteries

// Parameters
double P_max = 2.0; // Maximum power output per turbine (MW)
double SoC_min = 0.2; // Minimum State of Charge (SoC)
double SoC_max = 0.8; // Maximum State of Charge (SoC)
double P_storage_max = 1.0; // Maximum power input/output of battery (MW)

// Wind speed profile simulation
double wind_speed_profile(int t) {
    return 10.0 + 5.0 * sin(0.1 * t) + 2.0 * cos(0.05 * t); // Simulated wind speed (m/s)
}

// Turbine power curve approximation (simple quadratic model)
double turbine_power(double wind_speed) {
    if (wind_speed < 3.0 || wind_speed > 25.0) return 0.0; // Cut-in and cut-out speeds
    return P_max * pow(wind_speed / 15.0, 3.0); // Simplified power curve
}

// MPC Controller
void mpc_control(VectorXd& X, VectorXd& U, const MatrixXd& A, const MatrixXd& B, const VectorXd& c,
                 const VectorXd& lb_x, const VectorXd& ub_x, const VectorXd& lb_u, const VectorXd& ub_u) {
    // Decision variables: state trajectory (X) and control trajectory (U)
    MatrixXd Q = MatrixXd::Identity(num_turbines + num_batteries, num_turbines + num_batteries);
    MatrixXd R = MatrixXd::Identity(num_turbines, num_turbines);

    // Objective function coefficients
    VectorXd q = -Q * c;
    MatrixXd H = 2 * Q + R.replicate(num_turbines + num_batteries, 1).asDiagonal();

    // Constraints matrix
    MatrixXd G = MatrixXd::Zero(N * (num_turbines + num_batteries + num_turbines), 
                                N * (num_turbines + num_batteries) + num_turbines);
    VectorXd h = VectorXd::Zero(N * (num_turbines + num_batteries + num_turbines));

    // Initial state constraint
    G.block(0, 0, num_turbines + num_batteries, num_turbines + num_batteries) = MatrixXd::Identity(num_turbines + num_batteries, num_turbines + num_batteries);
    h.head(num_turbines + num_batteries) = X;

    // Dynamics constraints
    for (int k = 0; k < N; ++k) {
        G.block(k * (num_turbines + num_batteries) + num_turbines + num_batteries, 
                k * (num_turbines + num_batteries), 
                num_turbines + num_batteries, 
                num_turbines + num_batteries) = -MatrixXd::Identity(num_turbines + num_batteries, num_turbines + num_batteries);
        G.block(k * (num_turbines + num_batteries) + num_turbines + num_batteries, 
                k * (num_turbines + num_batteries) + (num_turbines + num_batteries) * N, 
                num_turbines + num_batteries, 
                num_turbines) = B;
        G.block(k * (num_turbines + num_batteries) + num_turbines + num_batteries, 
                (k + 1) * (num_turbines + num_batteries), 
                num_turbines + num_batteries, 
                num_turbines + num_batteries) = A;
        h.segment(k * (num_turbines + num_batteries) + num_turbines + num_batteries, num_turbines + num_batteries) = -A * X.tail(num_turbines + num_batteries);
    }

    // State bounds
    for (int k = 0; k <= N; ++k) {
        G.block(k * (num_turbines + num_batteries), 
                k * (num_turbines + num_batteries), 
                num_turbines + num_batteries, 
                num_turbines + num_batteries) = -MatrixXd::Identity(num_turbines + num_batteries, num_turbines + num_batteries);
        G.block((k + 1) * (num_turbines + num_batteries) - (num_turbines + num_batteries), 
                k * (num_turbines + num_batteries), 
                num_turbines + num_batteries, 
                num_turbines + num_batteries) = MatrixXd::Identity(num_turbines + num_batteries, num_turbines + num_batteries);
        h.segment(k * (num_turbines + num_batteries), num_turbines + num_batteries) = -lb_x;
        h.segment((k + 1) * (num_turbines + num_batteries) - (num_turbines + num_batteries), num_turbines + num_batteries) = ub_x;
    }

    // Control bounds
    for (int k = 0; k < N; ++k) {
        G.block(N * (num_turbines + num_batteries) + k * num_turbines, 
                k * (num_turbines + num_batteries) + (num_turbines + num_batteries) * N, 
                num_turbines, 
                num_turbines) = -MatrixXd::Identity(num_turbines, num_turbines);
        G.block(N * (num_turbines + num_batteries) + k * num_turbines + num_turbines, 
                k * (num_turbines + num_batteries) + (num_turbines + num_batteries) * N, 
                num_turbines, 
                num_turbines) = MatrixXd::Identity(num_turbines, num_turbines);
        h.segment(N * (num_turbines + num_batteries) + k * num_turbines, num_turbines) = -lb_u;
        h.segment(N * (num_turbines + num_batteries) + k * num_turbines + num_turbines, num_turbines) = ub_u;
    }

    // Solve the QP problem
    SparseMatrix<double> H_sparse(H.sparseView());
    SparseLU<SparseMatrix<double>> solver;
    solver.analyzePattern(H_sparse);
    solver.factorize(H_sparse);
    VectorXd b = H * X.tail(num_turbines + num_batteries) + q;
    VectorXd solution = solver.solve(b);

    // Extract control inputs
    U = solution.tail(num_turbines * N);
}

int main() {
    // Initial conditions
    VectorXd X(num_turbines + num_batteries);
    X.setConstant(0.5); // Initial SoC for batteries

    // Control inputs
    VectorXd U(num_turbines * N);

    // Dynamic model matrices
    MatrixXd A = MatrixXd::Identity(num_turbines + num_batteries, num_turbines + num_batteries);
    MatrixXd B = MatrixXd::Zero(num_turbines + num_batteries, num_turbines);
    for (int i = 0; i < num_turbines; ++i) {
        B(i, i) = dt * 0.01; // Example gain for simplicity
    }
    for (int i = num_turbines; i < num_turbines + num_batteries; ++i) {
        B(i, i - num_turbines) = dt * 0.01; // Example gain for simplicity
    }

    // State and control bounds
    VectorXd lb_x(num_turbines + num_batteries);
    VectorXd ub_x(num_turbines + num_batteries);
    VectorXd lb_u(num_turbines);
    VectorXd ub_u(num_turbines);
    lb_x.setConstant(SoC_min);
    ub_x.setConstant(SoC_max);
    lb_u.setConstant(-P_storage_max);
    ub_u.setConstant(P_storage_max);

    // Desired power output (setpoint)
    VectorXd c(num_turbines + num_batteries);
    c.setConstant(0.6); // Desired SoC for batteries

    // Simulation loop
    vector<double> time_points;
    vector<vector<double>> power_outputs(num_turbines);
    vector<vector<double>> soCs(num_batteries);
    vector<vector<double>> control_inputs(num_turbines);
    vector<double> load_demands;

    for (double t = 0.0; t <= 100.0; t += dt) {
        // Measure wind speed and calculate power outputs
        double wind_speed = wind_speed_profile(static_cast<int>(t));
        VectorXd power_outputs_current(num_turbines);
        for (int i = 0; i < num_turbines; ++i) {
            power_outputs_current(i) = turbine_power(wind_speed);
        }

        // Load demand profile
        double load_demand = 10.0 + 5.0 * sin(0.1 * t); // Simulated load demand (MW)
        load_demands.push_back(load_demand);

        // Apply MPC control
        mpc_control(X, U, A, B, c, lb_x, ub_x, lb_u, ub_u);

        // Update states using the first control input
        for (int i = 0; i < num_turbines; ++i) {
            X(i) += dt * 0.01 * U(i); // Example gain for simplicity
        }
        for (int i = num_turbines; i < num_turbines + num_batteries; ++i) {
            X(i) += dt * 0.01 * U(i - num_turbines); // Example gain for simplicity
        }

        // Store data for plotting
        time_points.push_back(t);
        for (int i = 0; i < num_turbines; ++i) {
            power_outputs[i].push_back(power_outputs_current(i));
            control_inputs[i].push_back(U(i));
        }
        for (int i = 0; i < num_batteries; ++i) {
            soCs[i].push_back(X(i + num_turbines));
        }

        // Update plot
        if (static_cast<int>(t) % 10 == 0) {
            plt::clf();
            plt::plot(time_points, power_outputs[0], label="Turbine 1 Power Output");
            plt::plot(time_points, power_outputs[1], label="Turbine 2 Power Output");
            plt::plot(time_points, power_outputs[2], label="Turbine 3 Power Output");
            plt::plot(time_points, power_outputs[3], label="Turbine 4 Power Output");
            plt::plot(time_points, power_outputs[4], label="Turbine 5 Power Output");
            plt::plot(time_points, soCs[0], label="Battery SoC", linestyle="--");
            plt::plot(time_points, load_demands, label="Load Demand", color="red", linestyle="-.");
            plt::xlabel("Time (s)");
            plt::ylabel("Power Output (MW) / SoC");
            plt::title("Wind Farm Operation with MPC Control");
            plt::legend();
            plt::pause(0.1);
        }
    }

    plt::show();
    return 0;
}



