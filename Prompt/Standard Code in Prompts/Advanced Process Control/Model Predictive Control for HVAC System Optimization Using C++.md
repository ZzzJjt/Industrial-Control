```
#include <iostream>
#include <vector>

// Simplified HVAC system model
class HVACModel {
public:
    HVACModel(double initTemp, double initHumid, double extTemp, double occLevel)
        : temp(initTemp), humidity(initHumid), externalTemp(extTemp), occupancyLevel(occLevel) {}

    // Update the state of the HVAC system
    void update(double heatPower, double coolPower, double humidPower, double dehumidPower) {
        // Simplified dynamics model
        temp += heatPower - coolPower + externalTemp * 0.1;
        humidity += humidPower - dehumidPower + occupancyLevel * 0.05;
    }

    // Get current state
    std::pair<double, double> getState() const {
        return std::make_pair(temp, humidity);
    }

private:
    double temp;          // Current temperature
    double humidity;      // Current humidity
    double externalTemp;  // External temperature
    double occupancyLevel;// Occupancy level
};
#include <Eigen/Dense> // For matrix operations
#include <unsupported/Eigen/MatrixFunctions> // For exponential matrix

using namespace Eigen;

class MPC {
public:
    MPC(const MatrixXd& A, const MatrixXd& B, int predictionHorizon, int controlHorizon)
        : A(A), B(B), N(predictionHorizon), M(controlHorizon) {}

    // Solve the MPC problem
    VectorXd solveMPC(const VectorXd& x, const VectorXd& r) {
        // Simplified QP formulation
        // This is a placeholder for the actual QP solver call
        // In practice, you'd use a library like qpOASES or OSQP
        VectorXd u(N); // Placeholder solution
        return u;
    }

private:
    MatrixXd A, B; // System matrices
    int N, M;      // Prediction and control horizons
};
int main() {
    // Initial conditions
    double initTemp = 22.0; // Initial temperature in degrees Celsius
    double initHumid = 50.0; // Initial humidity in percentage
    double extTemp = 25.0; // External temperature
    double occLevel = 0.5; // Occupancy level

    // Create HVAC model
    HVACModel hvac(initTemp, initHumid, extTemp, occLevel);

    // Define system matrices (simplified)
    MatrixXd A(2, 2);
    A << 1.0, 0.0,
         0.0, 1.0;
    MatrixXd B(2, 4);
    B << 1.0, -1.0, 0.0, 0.0,
         0.0, 0.0, 1.0, -1.0;

    // Create MPC controller
    MPC mpc(A, B, 10, 3);

    // Simulation parameters
    int numSteps = 100; // Number of simulation steps
    double dt = 0.1; // Time step

    // Simulation loop
    for (int t = 0; t < numSteps; ++t) {
        // Get current state
        auto state = hvac.getState();
        VectorXd x(2);
        x << state.first, state.second;

        // Reference values (desired temperature and humidity)
        VectorXd r(2);
        r << 23.0, 45.0;

        // Solve MPC
        VectorXd u = mpc.solveMPC(x, r);

        // Update HVAC system
        hvac.update(u(0), u(1), u(2), u(3));

        // Print current state
        std::cout << "Step: " << t << ", Temp: " << state.first << ", Humidity: " << state.second << std::endl;
    }

    return 0;
}
```
