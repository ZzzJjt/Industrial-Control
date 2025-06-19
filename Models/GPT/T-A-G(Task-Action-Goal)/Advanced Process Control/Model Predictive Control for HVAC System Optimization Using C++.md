#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>

// HVAC dynamics model (simplified)
struct HVACState {
    double temperature;
    double humidity;
};

struct HVACInput {
    double heating_power;
    double dehumidifier_power;
};

struct ExternalConditions {
    double outdoor_temp;
    double occupancy_level;
};

// Simulate one time step of HVAC dynamics
HVACState simulate_step(const HVACState& current, const HVACInput& input, const ExternalConditions& external) {
    double temp_gain = 0.1 * input.heating_power - 0.05 * (current.temperature - external.outdoor_temp) - 0.01 * external.occupancy_level;
    double humidity_gain = -0.1 * input.dehumidifier_power + 0.02 * external.occupancy_level;

    HVACState next;
    next.temperature = current.temperature + temp_gain;
    next.humidity = current.humidity + humidity_gain;
    return next;
}

// Cost function to minimize (temperature deviation + humidity + energy use)
double cost_function(const HVACState& state, const HVACInput& input, double temp_setpoint, double hum_max) {
    double temp_error = std::pow(state.temperature - temp_setpoint, 2);
    double hum_penalty = std::max(0.0, state.humidity - hum_max);
    double energy_cost = 0.05 * (std::pow(input.heating_power, 2) + std::pow(input.dehumidifier_power, 2));
    return temp_error + 10 * hum_penalty + energy_cost;
}

// Naive MPC: Test all input combinations in a limited range
HVACInput mpc_control(const HVACState& current, const ExternalConditions& external, double temp_setpoint, double hum_max) {
    HVACInput best_input;
    double best_cost = 1e9;

    for (double heat = 0; heat <= 5; heat += 0.5) {
        for (double dehum = 0; dehum <= 5; dehum += 0.5) {
            HVACInput test_input = {heat, dehum};
            HVACState predicted = simulate_step(current, test_input, external);
            double cost = cost_function(predicted, test_input, temp_setpoint, hum_max);
            if (cost < best_cost) {
                best_cost = cost;
                best_input = test_input;
            }
        }
    }
    return best_input;
}

int main() {
    HVACState state = {22.0, 0.50}; // Initial indoor temperature and RH
    double temp_setpoint = 24.0;
    double hum_max = 0.60;

    std::vector<ExternalConditions> forecast = {
        {30.0, 2}, {29.0, 3}, {28.0, 4}, {27.0, 2}, {26.0, 1}
    };

    std::cout << "Time\tTemp\tHumidity\tHeat\tDehum" << std::endl;
    for (size_t t = 0; t < forecast.size(); ++t) {
        HVACInput input = mpc_control(state, forecast[t], temp_setpoint, hum_max);
        state = simulate_step(state, input, forecast[t]);
        std::cout << t << "\t" << state.temperature << "\t" << state.humidity
                  << "\t" << input.heating_power << "\t" << input.dehumidifier_power << std::endl;
    }
    return 0;
}
