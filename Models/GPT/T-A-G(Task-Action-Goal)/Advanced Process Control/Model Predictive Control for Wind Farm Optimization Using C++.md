#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>
#include <random>

// Wind farm system state
struct WindFarmState {
    double turbine_speed;  // RPM
    double storage_level;  // MWh
};

struct WindFarmInput {
    double pitch_angle;    // degrees
    double charge_rate;    // MWh/h
};

// Wind farm dynamics (simplified)
WindFarmState simulate_step(const WindFarmState& current, const WindFarmInput& input, double wind_speed) {
    // Turbine power extraction depends on pitch angle and wind speed
    double power_output = std::max(0.0, wind_speed * (1.0 - input.pitch_angle / 90.0));
    power_output = std::min(power_output, 5.0); // Max turbine output (MW)

    // Update storage
    double new_storage = current.storage_level + input.charge_rate - power_output * 0.1;
    new_storage = std::clamp(new_storage, 0.0, 10.0);

    // Update turbine speed
    double new_speed = current.turbine_speed + 0.05 * (wind_speed - current.turbine_speed) - 0.02 * input.pitch_angle;

    return {new_speed, new_storage};
}

// Cost function for MPC
double cost_function(const WindFarmState& predicted, const WindFarmInput& input, double target_speed, double target_storage) {
    double speed_error = std::pow(predicted.turbine_speed - target_speed, 2);
    double storage_error = std::pow(predicted.storage_level - target_storage, 2);
    double control_effort = 0.1 * (std::pow(input.pitch_angle, 2) + std::pow(input.charge_rate, 2));
    return speed_error + storage_error + control_effort;
}

// MPC controller (brute-force approach)
WindFarmInput mpc_control(const WindFarmState& current, double wind_speed, double target_speed, double target_storage) {
    double best_cost = 1e9;
    WindFarmInput best_input = {0, 0};

    for (double pitch = 0.0; pitch <= 30.0; pitch += 5.0) {
        for (double charge = -1.0; charge <= 1.0; charge += 0.5) {
            WindFarmInput input = {pitch, charge};
            WindFarmState pred = simulate_step(current, input, wind_speed);
            double cost = cost_function(pred, input, target_speed, target_storage);
            if (cost < best_cost) {
                best_cost = cost;
                best_input = input;
            }
        }
    }
    return best_input;
}

int main() {
    WindFarmState state = {10.0, 5.0};
    std::default_random_engine generator;
    std::normal_distribution<double> wind_dist(12.0, 2.0);

    double target_speed = 15.0;
    double target_storage = 6.0;

    std::cout << "Time\tWind\tSpeed\tStorage\tPitch\tCharge" << std::endl;
    for (int t = 0; t < 20; ++t) {
        double wind = std::clamp(wind_dist(generator), 6.0, 18.0);
        WindFarmInput input = mpc_control(state, wind, target_speed, target_storage);
        state = simulate_step(state, input, wind);

        std::cout << t << "\t" << wind << "\t" << state.turbine_speed << "\t"
                  << state.storage_level << "\t" << input.pitch_angle << "\t" << input.charge_rate << std::endl;
    }
    return 0;
}
