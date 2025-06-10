#include <iostream>
#include <vector>
#include <cmath>
#include <iomanip>

// Define simulation parameters
const int TIME_HORIZON = 24; // hours
const float OUTDOOR_TEMP_DAY = 35.0; // Celsius
const float OUTDOOR_TEMP_NIGHT = 22.0;
const float TEMP_SETPOINT = 24.0;
const float HUMIDITY_SETPOINT = 50.0;
const float TEMP_WEIGHT = 1.0;
const float HUMIDITY_WEIGHT = 0.5;
const float ENERGY_WEIGHT = 0.1;

// Structure to hold system state
struct HVACState {
    float indoor_temp;
    float indoor_humidity;
    float energy_usage;
};

// Predict occupancy level
int get_occupancy(int hour) {
    return (hour >= 8 && hour <= 18) ? 1 : 0;
}

// Predict external temperature
float get_outdoor_temp(int hour) {
    return (hour >= 6 && hour <= 18) ? OUTDOOR_TEMP_DAY : OUTDOOR_TEMP_NIGHT;
}

// Simple MPC controller
HVACState run_mpc(const HVACState& current, float outdoor_temp, int occupancy) {
    HVACState new_state = current;
    float control_temp = TEMP_SETPOINT;
    float control_humidity = HUMIDITY_SETPOINT;

    float temp_error = control_temp - current.indoor_temp;
    float humidity_error = control_humidity - current.indoor_humidity;

    // Simple proportional adjustment
    new_state.indoor_temp += 0.3 * temp_error - 0.1 * (outdoor_temp - current.indoor_temp);
    new_state.indoor_humidity += 0.2 * humidity_error - 0.05 * occupancy;

    // Estimate energy cost
    new_state.energy_usage = std::abs(temp_error) * TEMP_WEIGHT
                           + std::abs(humidity_error) * HUMIDITY_WEIGHT
                           + occupancy * ENERGY_WEIGHT;

    return new_state;
}

int main() {
    HVACState state = {28.0, 60.0, 0.0};

    std::cout << std::fixed << std::setprecision(2);
    std::cout << "Hour\tTemp(C)\tHumidity(%)\tEnergy(kWh)\n";

    for (int hour = 0; hour < TIME_HORIZON; ++hour) {
        float outdoor_temp = get_outdoor_temp(hour);
        int occupancy = get_occupancy(hour);

        state = run_mpc(state, outdoor_temp, occupancy);

        std::cout << hour << "\t" << state.indoor_temp << "\t"
                  << state.indoor_humidity << "\t\t" << state.energy_usage << "\n";
    }

    return 0;
}
