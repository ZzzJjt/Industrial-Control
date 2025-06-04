#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

// Simulation parameters
const int N = 10;                      // Prediction horizon
const int T_total = 100;              // Total simulation time
const double dt = 1.0;                // Time step
const double P_max = 1000.0;          // Max power output (kW)
const double storage_max = 500.0;     // Max energy storage (kWh)
const double eta = 0.9;               // Storage efficiency

// Simple wind-to-power conversion (nonlinear)
double windToPower(double wind_speed) {
    if (wind_speed < 3.0 || wind_speed > 25.0) return 0.0;
    if (wind_speed <= 15.0)
        return P_max * pow(wind_speed / 15.0, 3);
    return P_max;
}

// MPC placeholder: returns control input (storage dispatch) for current time step
double mpcController(double power_pred, double grid_demand, double storage_level) {
    double error = grid_demand - power_pred;
    double dispatch = std::clamp(error / eta, -100.0, 100.0); // Dispatch from/to storage
    double new_level = storage_level - dispatch * dt;

    if (new_level > storage_max) dispatch = (storage_level - storage_max) / dt;
    if (new_level < 0.0) dispatch = storage_level / dt;

    return dispatch;
}

int main() {
    std::vector<double> wind_speed(T_total);
    std::vector<double> power_output(T_total);
    std::vector<double> grid_demand(T_total);
    std::vector<double> storage_level(T_total);
    std::vector<double> dispatch(T_total);

    // Initialize variables
    storage_level[0] = 250.0; // Start at half capacity

    // Generate wind speed and demand profiles
    for (int t = 0; t < T_total; ++t) {
        wind_speed[t] = 8.0 + 4.0 * sin(0.1 * t);          // Varying wind
        grid_demand[t] = 600.0 + 100.0 * sin(0.05 * t);    // Varying demand
    }

    // Simulation loop
    for (int t = 0; t < T_total; ++t) {
        double power = windToPower(wind_speed[t]);
        double u = mpcController(power, grid_demand[t], storage_level[t]);

        dispatch[t] = u;
        power_output[t] = power + eta * u;
        if (t < T_total - 1)
            storage_level[t + 1] = std::clamp(storage_level[t] - u * dt, 0.0, storage_max);

        // Print results
        std::cout << "t=" << t << "  Wind=" << wind_speed[t]
                  << "  Power=" << power
                  << "  Demand=" << grid_demand[t]
                  << "  Dispatch=" << u
                  << "  Storage=" << storage_level[t] << std::endl;
    }

    return 0;
}
