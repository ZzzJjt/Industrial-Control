#include <iostream>
#include <vector>
#include <cmath>

using namespace std;

// Constants
const double dt = 1.0; // Time step (minutes)
const int horizon = 60; // Prediction horizon (time steps)

// Parameters
double T_setpoint = 22.0; // Desired temperature (Celsius)
double H_setpoint = 50.0; // Desired humidity (%)
double T_outdoor_initial = 15.0; // Initial outdoor temperature (Celsius)
double H_outdoor_initial = 45.0; // Initial outdoor humidity (%)
double occupancy_initial = 1.0; // Initial occupancy level (normalized between 0 and 1)
double alpha_T = 0.05; // Temperature change coefficient
double alpha_H = 0.03; // Humidity change coefficient
double beta_T = 0.1; // Outdoor influence on temperature
double beta_H = 0.08; // Outdoor influence on humidity
double gamma_T = 0.2; // Occupancy influence on temperature
double gamma_H = 0.15; // Occupancy influence on humidity

// State variables
struct State {
    double T_indoor;
    double H_indoor;
};

// Control inputs
struct Control {
    double Q_heating_cooling; // Heating/Cooling power (kW)
    double Q_humid_dehumid; // Humidification/Dehumidification power (kW)
};

// Dynamic model of the HVAC system
State dynamic_model(State state, Control control, double T_outdoor, double H_outdoor, double occupancy) {
    State next_state;
    next_state.T_indoor = state.T_indoor + dt * (alpha_T * (T_setpoint - state.T_indoor) +
                                                beta_T * (T_outdoor - state.T_indoor) +
                                                gamma_T * occupancy -
                                                control.Q_heating_cooling);
    next_state.H_indoor = state.H_indoor + dt * (alpha_H * (H_setpoint - state.H_indoor) +
                                                beta_H * (H_outdoor - state.H_indoor) +
                                                gamma_H * occupancy -
                                                control.Q_humid_dehumid);
    return next_state;
}

// Objective function: minimize deviation from setpoints and energy consumption
double objective_function(const vector<State>& states, const vector<Control>& controls) {
    double cost = 0.0;
    for (size_t i = 0; i < states.size(); ++i) {
        cost += pow(states[i].T_indoor - T_setpoint, 2) + pow(states[i].H_indoor - H_setpoint, 2) + 
                0.1 * (controls[i].Q_heating_cooling * controls[i].Q_heating_cooling + 
                       controls[i].Q_humid_dehumid * controls[i].Q_humid_dehumid);
    }
    return cost;
}

// MPC controller
Control mpc_control(State current_state, double T_outdoor, double H_outdoor, double occupancy) {
    Control best_control;
    double min_cost = numeric_limits<double>::max();

    // Try different control actions
    for (double q_heat_cool = -1.0; q_heat_cool <= 1.0; q_heat_cool += 0.2) {
        for (double q_humid_dehumid = -1.0; q_humid_dehumid <= 1.0; q_humid_dehumid += 0.2) {
            Control trial_control = {q_heat_cool, q_humid_dehumid};
            vector<State> predicted_states(horizon);
            predicted_states[0] = current_state;

            // Simulate future states
            for (int t = 1; t < horizon; ++t) {
                predicted_states[t] = dynamic_model(predicted_states[t-1], trial_control, T_outdoor, H_outdoor, occupancy);
            }

            // Evaluate cost
            double cost = objective_function(predicted_states, vector<Control>(horizon, trial_control));
            if (cost < min_cost) {
                min_cost = cost;
                best_control = trial_control;
            }
        }
    }
    return best_control;
}

int main() {
    // Initial conditions
    State state = {22.0, 50.0}; // Start at setpoints
    double T_outdoor = T_outdoor_initial;
    double H_outdoor = H_outdoor_initial;
    double occupancy = occupancy_initial;

    // Simulation loop
    vector<double> time_points;
    vector<double> T_indoor_history;
    vector<double> H_indoor_history;
    vector<double> Q_heat_cool_history;
    vector<double> Q_humid_dehumid_history;

    for (double t = 0.0; t <= 24 * 60; t += dt) {
        // Measure external conditions and occupancy (simple sinusoidal variation for demonstration)
        T_outdoor = T_outdoor_initial + 5.0 * sin(0.01 * t);
        H_outdoor = H_outdoor_initial + 5.0 * cos(0.01 * t);
        occupancy = occupancy_initial + 0.5 * sin(0.02 * t);

        // Apply MPC control
        Control control = mpc_control(state, T_outdoor, H_outdoor, occupancy);

        // Update state using the control action
        state = dynamic_model(state, control, T_outdoor, H_outdoor, occupancy);

        // Store data for plotting
        time_points.push_back(t);
        T_indoor_history.push_back(state.T_indoor);
        H_indoor_history.push_back(state.H_indoor);
        Q_heat_cool_history.push_back(control.Q_heating_cooling);
        Q_humid_dehumid_history.push_back(control.Q_humid_dehumid);
    }

    // Plot results (simple console output for demonstration)
    cout << "Time\tT_indoor\tH_indoor\tQ_heat_cool\tQ_humid_dehumid\n";
    for (size_t i = 0; i < time_points.size(); ++i) {
        cout << time_points[i] << "\t" << T_indoor_history[i] << "\t" << H_indoor_history[i] << "\t"
             << Q_heat_cool_history[i] << "\t" << Q_humid_dehumid_history[i] << "\n";
    }

    return 0;
}



