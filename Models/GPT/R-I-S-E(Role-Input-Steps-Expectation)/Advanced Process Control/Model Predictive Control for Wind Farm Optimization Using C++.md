// windfarm_mpc_control.cpp
// Model Predictive Control for Wind Farm with Energy Storage

#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

const double dt = 1.0;                      // Time step [s]
const int horizon = 10;                    // Prediction horizon
const double P_max = 2.5;                  // Max turbine output [MW]
const double SOC_max = 10.0;               // Max state of charge [MWh]
const double SOC_min = 1.0;                // Min state of charge [MWh]
const double charge_rate_max = 0.5;        // Max charge/discharge rate [MW]
const double efficiency = 0.95;            // Round-trip efficiency of storage

struct SystemState {
    double wind_speed;   // [m/s]
    double turbine_power; // [MW]
    double soc;           // [MWh]
};

// Simplified wind turbine power curve
double windTurbinePower(double wind_speed) {
    if (wind_speed < 3.0 || wind_speed > 25.0)
        return 0.0;
    else if (wind_speed >= 12.0)
        return P_max;
    else
        return P_max * std::pow((wind_speed / 12.0), 3);
}

// Predict future wind speeds (mock function)
std::vector<double> predictWind(const double current_speed) {
    std::vector<double> predictions(horizon);
    for (int i = 0; i < horizon; ++i)
        predictions[i] = current_speed + 0.5 * std::sin(0.1 * i); // fluctuation
    return predictions;
}

// MPC controller
void MPCController(SystemState &state, double load_demand) {
    std::vector<double> wind_forecast = predictWind(state.wind_speed);

    double best_cost = 1e9;
    double best_charge = 0.0;
    double best_turbine = 0.0;

    // Search over control combinations
    for (double turbine = 0.0; turbine <= P_max; turbine += 0.1) {
        for (double charge = -charge_rate_max; charge <= charge_rate_max; charge += 0.1) {
            double soc_sim = state.soc;
            double cost = 0.0;
            bool valid = true;

            for (int k = 0; k < horizon; ++k) {
                double wind_pow = windTurbinePower(wind_forecast[k]);
                double power_out = std::min(wind_pow, turbine);
                soc_sim += efficiency * charge * dt;
                if (soc_sim < SOC_min || soc_sim > SOC_max) {
                    valid = false;
                    break;
                }
                double net_power = power_out + charge;
                cost += std::pow(net_power - load_demand, 2);
            }

            if (valid && cost < best_cost) {
                best_cost = cost;
                best_charge = charge;
                best_turbine = turbine;
            }
        }
    }

    // Apply control
    state.turbine_power = best_turbine;
    state.soc += best_charge * dt;
    std::cout << "MPC control applied: TurbinePower=" << best_turbine
              << " MW, ChargeRate=" << best_charge << " MW, SOC=" << state.soc << " MWh\n";
}

int main() {
    SystemState state{10.0, 0.0, 5.0};
    const int total_steps = 50;
    double load_demand = 2.0; // Constant load for demo

    for (int t = 0; t < total_steps; ++t) {
        std::cout << "Time step " << t << ":\n";
        MPCController(state, load_demand);
        state.wind_speed += 0.2 * std::sin(0.2 * t); // mock wind disturbance
    }

    return 0;
}
