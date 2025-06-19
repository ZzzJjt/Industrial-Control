#include <iostream>
#include <vector>
#include <cmath>
#include <Eigen/Dense>

using namespace std;
using namespace Eigen;

// Constants
const double Ts = 0.1; // Sampling time [seconds]
const int Np = 10;     // Prediction horizon
const int Nu = 2;      // Number of control inputs (blade pitch angle, battery charge/discharge rate)
const int Nx = 4;      // Number of states (wind speed, turbine power output, battery SOC, grid frequency)

// Dynamic Model: Wind Turbine Power Curve
double turbine_power(double ws, double pitch_angle) {
    // Simple cubic power curve model
    double rated_ws = 12.0; // Rated wind speed [m/s]
    double cut_in_ws = 3.0; // Cut-in wind speed [m/s]
    double cut_out_ws = 25.0; // Cut-out wind speed [m/s]
    double rated_power = 2.0; // Rated power [MW]

    if (ws < cut_in_ws || ws > cut_out_ws) {
        return 0.0;
    } else if (ws >= cut_in_ws && ws <= rated_ws) {
        return rated_power * pow((ws - cut_in_ws) / (rated_ws - cut_in_ws), 3);
    } else {
        return rated_power * pow(1.0 - (ws - rated_ws) / (cut_out_ws - rated_ws), 2);
    }
}

// Battery Dynamics
double battery_dynamics(double soc, double charge_rate) {
    double max_soc = 1.0; // Maximum State of Charge
    double min_soc = 0.0; // Minimum State of Charge
    double charging_efficiency = 0.95;
    double discharging_efficiency = 0.95;

    if (charge_rate > 0) {
        return charging_efficiency * charge_rate;
    } else {
        return discharging_efficiency * charge_rate;
    }
}

// Motion Model: Update state based on control inputs and disturbances
VectorXd motion_model(const VectorXd& x, const VectorXd& u, double ws) {
    double pitch_angle = u[0];
    double battery_charge_rate = u[1];

    double turbine_pwr = turbine_power(ws, pitch_angle);
    double soc_change = battery_dynamics(x[2], battery_charge_rate);

    VectorXd dx(Nx);
    dx << 0.0, // Wind speed does not change dynamically in this simple model
          turbine_pwr,
          soc_change,
          0.0; // Grid frequency does not change dynamically in this simple model

    return dx;
}

// Initialize matrices for QP
MatrixXd Q = MatrixXd::Identity(Nx, Nx); // State weight matrix
MatrixXd R = MatrixXd::Identity(Nu, Nu); // Input weight matrix

// Define operational constraints
VectorXd u_min(Nu);
VectorXd u_max(Nu);
VectorXd x_min(Nx);
VectorXd x_max(Nx);

void initializeConstraints() {
    u_min << -0.1, -0.5; // Minimum blade pitch angle, battery discharge rate
    u_max << 0.1, 0.5;   // Maximum blade pitch angle, battery charge rate
    x_min << 0.0, 0.0, 0.0, 49.8; // Minimum wind speed, turbine power, battery SOC, grid frequency
    x_max << 25.0, 2.0, 1.0, 50.2; // Maximum wind speed, turbine power, battery SOC, grid frequency
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
    MPCController(int nx, int nu, int np, const MatrixXd& Q, const MatrixXd& R, const VectorXd& u_min, const VectorXd& u_max, const VectorXd& x_min, const VectorXd& x_max)
        : nx(nx), nu(nu), np(np), Q(Q), R(R), u_min(u_min), u_max(u_max), x_min(x_min), x_max(x_max) {}

    VectorXd computeControl(const VectorXd& x0, double ws) {
        int n = nx * np + nu * (np - 1);
        MatrixXd H = MatrixXd::Zero(n, n);
        VectorXd f = VectorXd::Zero(n);
        MatrixXd Aeq = MatrixXd::Zero(nx, n);
        VectorXd beq = VectorXd::Zero(nx);
        MatrixXd Ain = MatrixXd::Zero(2 * nu * (np - 1) + 2 * nx * np, n);
        VectorXd bin = VectorXd::Zero(2 * nu * (np - 1) + 2 * nx * np);
        VectorXd lb = VectorXd::Zero(n);
        VectorXd ub = VectorXd::Zero(n);

        // Construct H and f
        for (int k = 0; k < np; ++k) {
            H.block(k * nx, k * nx, nx, nx) += Q;
        }
        for (int k = 0; k < np - 1; ++k) {
            H.block(nx * np + k * nu, nx * np + k * nu, nu, nu) += R;
        }

        // Construct Aeq and beq
        Aeq.block(0, 0, nx, nx) = MatrixXd::Identity(nx, nx);
        Aeq.block(0, nx, nx, nx * (np - 1)) = -MatrixXd::Identity(nx, nx);
        beq = -motion_model(x0, VectorXd::Zero(nu), ws);

        // Construct Ain and bin
        for (int k = 0; k < np - 1; ++k) {
            Ain.block(2 * k * nu, nx * np + k * nu, nu, nu) = MatrixXd::Identity(nu, nu);
            Ain.block((2 * k + 1) * nu, nx * np + k * nu, nu, nu) = -MatrixXd::Identity(nu, nu);
            bin.segment(2 * k * nu, nu) = u_min;
            bin.segment((2 * k + 1) * nu, nu) = -u_max;
        }
        for (int k = 0; k < np; ++k) {
            Ain.block(2 * nu * (np - 1) + 2 * k * nx, k * nx, nx, nx) = MatrixXd::Identity(nx, nx);
            Ain.block(2 * nu * (np - 1) + (2 * k + 1) * nx, k * nx, nx, nx) = -MatrixXd::Identity(nx, nx);
            bin.segment(2 * nu * (np - 1) + 2 * k * nx, nx) = x_min;
            bin.segment(2 * nu * (np - 1) + (2 * k + 1) * nx, nx) = -x_max;
        }

        // Solve QP
        VectorXd sol = solveQP(H, f, Aeq, beq, Ain, bin, lb, ub);

        // Extract first control input
        return sol.segment(nx * np, nu);
    }

private:
    int nx, nu, np;
    MatrixXd Q, R;
    VectorXd u_min, u_max, x_min, x_max;
};

int main() {
    // Initialize constraints
    initializeConstraints();

    // Initial state
    VectorXd x0(Nx);
    x0 << 10.0, 0.0, 0.5, 50.0; // Initial wind speed, turbine power, battery SOC, grid frequency

    // Create MPC controller
    MPCController mpc(Nx, Nu, Np, Q, R, u_min, u_max, x_min, x_max);

    // Simulation settings
    int simulation_steps = 100;
    vector<VectorXd> x_history(simulation_steps, VectorXd::Zero(Nx));
    vector<VectorXd> u_history(simulation_steps, VectorXd::Zero(Nu));

    // External disturbances
    vector<double> wind_speeds(simulation_steps);
    vector<double> load_demands(simulation_steps);

    // Generate random disturbances and load demands
    srand(time(0));
    for (int t = 0; t < simulation_steps; ++t) {
        wind_speeds[t] = 3.0 + static_cast<double>(rand()) / RAND_MAX * 22.0; // Random wind speed between 3 and 25 m/s
        load_demands[t] = 1.0 + static_cast<double>(rand()) / RAND_MAX * 1.5; // Random load demand between 1 and 2.5 MW
    }

    // Simulation loop
    for (int t = 0; t < simulation_steps; ++t) {
        // Compute control action
        VectorXd u = mpc.computeControl(x0, wind_speeds[t]);
        
        // Store history
        x_history[t] = x0;
        u_history[t] = u;

        // Apply control action and update state
        VectorXd dx = Ts * motion_model(x0, u, wind_speeds[t]);
        x0 += dx;

        // Adjust turbine power to match load demand
        double generated_power = x0[1];
        double net_power = generated_power + u[1]; // Generated power plus/minus battery charge/discharge

        // Print current state and control action
        cout << "Time: " << t * Ts << " s" << endl;
        cout << "State: Wind Speed = " << x0[0] << " m/s, Turbine Power = " << x0[1] << " MW, Battery SOC = " << x0[2] << ", Grid Frequency = " << x0[3] << " Hz" << endl;
        cout << "Control: Blade Pitch Angle = " << u[0] << " rad, Battery Charge Rate = " << u[1] << " MW" << endl;
        cout << "Wind Speed Disturbance: " << wind_speeds[t] << " m/s" << endl;
        cout << "Load Demand: " << load_demands[t] << " MW" << endl;
        cout << "Generated Power: " << generated_power << " MW, Net Power: " << net_power << " MW" << endl;
        cout << "----------------------------------------" << endl;
    }

    return 0;
}
