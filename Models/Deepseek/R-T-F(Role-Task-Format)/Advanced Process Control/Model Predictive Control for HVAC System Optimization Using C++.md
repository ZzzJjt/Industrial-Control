#include <iostream>
#include <vector>
#include <cmath>
#include <Eigen/Dense>

using namespace std;
using namespace Eigen;

// Constants
const double Ts = 1.0; // Sampling time [hours]
const int Np = 10;     // Prediction horizon
const int Nu = 2;      // Number of control inputs (Heater Power, Humidifier Power)
const int Nx = 3;      // Number of states (Temperature, Humidity, Energy Consumption)

// State transition matrix (A)
MatrixXd A(Nx, Nx);
// Input matrix (B)
MatrixXd B(Nx, Nu);
// Process noise covariance matrix (Q)
MatrixXd Q(Nx, Nx);
// Control weight matrix (R)
MatrixXd R(Nu, Nu);

// Initialize matrices
void initializeMatrices() {
    A << 0.95, 0.01, 0.01,
         0.01, 0.98, 0.005,
         0.01, 0.005, 0.99;
    B << 0.01, 0.001,
         0.001, 0.01,
         0.005, 0.005;
    Q << 1, 0, 0,
         0, 1, 0,
         0, 0, 0.1;
    R << 0.1, 0,
         0, 0.1;
}

// Quadratic Programming Solver using Gradient Descent
VectorXd solveQP(const MatrixXd& H, const VectorXd& f, const MatrixXd& Aeq, const VectorXd& beq, 
                 const MatrixXd& Ain, const VectorXd& bin, const VectorXd& lb, const VectorXd& ub) {
    int n = H.cols();
    VectorXd x = VectorXd::Zero(n);
    double alpha = 0.01; // Learning rate
    int max_iter = 1000;
    double tol = 1e-6;

    for (int iter = 0; iter < max_iter; ++iter) {
        VectorXd grad_H = H * x + f;
        VectorXd grad_Aeq = Aeq.transpose() * (Aeq * x - beq);
        VectorXd grad_Ain = Ain.transpose() * ((Ain * x - bin).cwiseMax(VectorXd::Zero(Ain.rows())));
        VectorXd grad_lb = (-lb.cwiseMin(x)).cwiseProduct((-lb.cwiseMin(x)).array().sign());
        VectorXd grad_ub = (ub.cwiseMax(x) - ub).cwiseProduct((ub.cwiseMax(x) - ub).array().sign());

        VectorXd gradient = grad_H + grad_Aeq + grad_Ain + grad_lb + grad_ub;

        if (gradient.norm() < tol) {
            break;
        }

        x -= alpha * gradient;
    }

    return x;
}

// MPC Controller
class MPCController {
public:
    MPCController(int nx, int nu, int np, const MatrixXd& A, const MatrixXd& B, const MatrixXd& Q, const MatrixXd& R)
        : nx(nx), nu(nu), np(np), A(A), B(B), Q(Q), R(R) {}

    VectorXd computeControl(const VectorXd& x0, const VectorXd& r) {
        int n = nx * np + nu * (np - 1);
        MatrixXd H = MatrixXd::Zero(n, n);
        VectorXd f = VectorXd::Zero(n);
        MatrixXd Aeq = MatrixXd::Zero(nx, n);
        VectorXd beq = VectorXd::Zero(nx);
        MatrixXd Ain = MatrixXd::Zero(2 * nu * (np - 1), n);
        VectorXd bin = VectorXd::Zero(2 * nu * (np - 1));
        VectorXd lb = VectorXd::Zero(n);
        VectorXd ub = VectorXd::Zero(n);

        // Construct H and f
        for (int k = 0; k < np; ++k) {
            H.block(k * nx, k * nx, nx, nx) += Q;
        }
        for (int k = 0; k < np - 1; ++k) {
            H.block(nx * np + k * nu, nx * np + k * nu, nu, nu) += R;
        }
        for (int k = 0; k < np; ++k) {
            f.segment(k * nx, nx) = -Q * r.segment(k * nx, nx);
        }
        for (int k = 0; k < np - 1; ++k) {
            f.segment(nx * np + k * nu, nu) = -R * VectorXd::Zero(nu);
        }

        // Construct Aeq and beq
        Aeq.block(0, 0, nx, nx) = MatrixXd::Identity(nx, nx) - A;
        Aeq.block(0, nx, nx, nx * (np - 1)) = -B;
        beq = -A * x0;

        // Construct Ain and bin
        for (int k = 0; k < np - 1; ++k) {
            Ain.block(2 * k * nu, nx * np + k * nu, nu, nu) = MatrixXd::Identity(nu, nu);
            Ain.block((2 * k + 1) * nu, nx * np + k * nu, nu, nu) = -MatrixXd::Identity(nu, nu);
            bin.segment(2 * k * nu, nu) = VectorXd::Zero(nu);
            bin.segment((2 * k + 1) * nu, nu) = VectorXd::Zero(nu);
        }

        // Construct lb and ub
        for (int k = 0; k < np - 1; ++k) {
            lb.segment(nx * np + k * nu, nu) = VectorXd::Constant(nu, 0.0); // Lower bound for u
            ub.segment(nx * np + k * nu, nu) = VectorXd::Constant(nu, 1.0); // Upper bound for u
        }

        // Solve QP
        VectorXd sol = solveQP(H, f, Aeq, beq, Ain, bin, lb, ub);

        // Extract first control input
        return sol.segment(nx * np, nu);
    }

private:
    int nx, nu, np;
    MatrixXd A, B, Q, R;
};

int main() {
    // Initialize matrices
    initializeMatrices();

    // Define setpoints
    VectorXd r(Nx * Np);
    for (int k = 0; k < Np; ++k) {
        r.segment(k * Nx, Nx) << 22.0, 50.0, 0.0; // Desired temperature, humidity, energy consumption
    }

    // Initial state
    VectorXd x0(Nx);
    x0 << 20.0, 45.0, 0.0; // Initial temperature, humidity, energy consumption

    // Create MPC controller
    MPCController mpc(Nx, Nu, Np, A, B, Q, R);

    // Simulation settings
    int simulation_steps = 100;
    vector<VectorXd> x_history(simulation_steps, VectorXd::Zero(Nx));
    vector<VectorXd> u_history(simulation_steps, VectorXd::Zero(Nu));

    // External disturbances
    vector<double> external_temp_disturbance(simulation_steps);
    vector<double> occupancy_levels(simulation_steps);

    // Generate random disturbances and occupancy levels
    srand(time(0));
    for (int t = 0; t < simulation_steps; ++t) {
        external_temp_disturbance[t] = 20.0 + static_cast<double>(rand()) / RAND_MAX * 10.0; // Random temp between 20 and 30
        occupancy_levels[t] = static_cast<double>(rand()) / RAND_MAX * 5.0; // Random occupancy between 0 and 5 people
    }

    // Simulation loop
    for (int t = 0; t < simulation_steps; ++t) {
        // Compute control action
        VectorXd u = mpc.computeControl(x0, r);

        // Store history
        x_history[t] = x0;
        u_history[t] = u;

        // Update setpoints based on occupancy levels
        r.segment(t % Np * Nx, Nx) << 22.0 + occupancy_levels[t], 50.0, 0.0;

        // Apply control action and update state
        VectorXd dx = A * x0 + B * u;
        dx[0] += external_temp_disturbance[t]; // Add external temperature disturbance

        x0 += Ts * dx;

        // Print current state and control action
        cout << "Time: " << t * Ts << " hours" << endl;
        cout << "State: Temp = " << x0[0] << " °C, Humidity = " << x0[1] << " %, Energy = " << x0[2] << " kWh" << endl;
        cout << "Control: Heater Power = " << u[0] << ", Humidifier Power = " << u[1] << endl;
        cout << "External Temp Disturbance: " << external_temp_disturbance[t] << " °C" << endl;
        cout << "Occupancy Level: " << occupancy_levels[t] << " people" << endl;
        cout << "----------------------------------------" << endl;
    }

    return 0;
}
