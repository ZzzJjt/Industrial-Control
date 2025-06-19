#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

const int HORIZON = 10;     // Prediction horizon
const double dt = 1.0;      // Time step [hr]
const double T_target = 22.0;  // Desired indoor temperature [°C]

// System parameters (simple model)
double a = 0.9;  // room thermal inertia
double b = 0.05; // HVAC effect
double c = 0.02; // external temp effect
double d = 0.01; // occupancy effect

// Cost weights
double weight_temp_error = 1.0;
double weight_energy_use = 0.1;

// Predict indoor temperature over horizon for a given control sequence
std::vector<double> predictTemp(double T0, const std::vector<double>& u,
                                const std::vector<double>& Text,
                                const std::vector<int>& occ) {
    std::vector<double> T(HORIZON + 1);
    T[0] = T0;
    for (int k = 0; k < HORIZON; ++k) {
        T[k + 1] = a * T[k] + b * u[k] + c * Text[k] + d * occ[k];
    }
    return T;
}

// Compute cost for a given control sequence
double computeCost(const std::vector<double>& T, const std::vector<double>& u) {
    double cost = 0.0;
    for (int k = 0; k < HORIZON; ++k) {
        double error = T[k] - T_target;
        cost += weight_temp_error * error * error + weight_energy_use * u[k] * u[k];
    }
    return cost;
}

// Optimize control sequence via brute-force (simple)
std::vector<double> optimizeMPC(double T0, const std::vector<double>& Text,
                                const std::vector<int>& occ) {
    double bestCost = 1e9;
    std::vector<double> bestU(HORIZON, 0.0);

    // Try different constant control inputs [0, 1, ..., 10]
    for (int hvac_power = 0; hvac_power <= 10; ++hvac_power) {
        std::vector<double> u(HORIZON, hvac_power);
        std::vector<double> T_pred = predictTemp(T0, u, Text, occ);
        double cost = computeCost(T_pred, u);
        if (cost < bestCost) {
            bestCost = cost;
            bestU = u;
        }
    }
    return bestU;
}

int main() {
    double Tin = 25.0;  // Initial indoor temperature
    std::vector<double> Text(HORIZON, 30.0);  // External temperature forecast
    std::vector<int> occ(HORIZON, 5);         // Occupancy forecast

    std::vector<double> u_opt = optimizeMPC(Tin, Text, occ);
    std::vector<double> T_pred = predictTemp(Tin, u_opt, Text, occ);

    std::cout << "Optimized HVAC Power (first step): " << u_opt[0] << "\n";
    std::cout << "Predicted Temperature Trajectory:\n";
    for (double T : T_pred) {
        std::cout << T << " ";
    }
    std::cout << "\n";

    return 0;
}
