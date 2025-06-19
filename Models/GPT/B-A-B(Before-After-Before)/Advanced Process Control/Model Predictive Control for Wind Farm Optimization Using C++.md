#include <iostream>
#include <vector>
#include <cmath>
#include <limits>

using namespace std;

// Simulation constants
const double dt = 1.0;  // time step (s)
const int SimTime = 60; // total simulation time
const int Horizon = 10; // MPC horizon

// Wind turbine model
double turbine_power_output(double wind_speed) {
    // Simplified power curve (cubic relationship until rated wind speed)
    if (wind_speed < 3.0) return 0.0;
    if (wind_speed > 15.0) return 1.0;  // normalized to 1 MW max
    return pow(wind_speed / 15.0, 3.0);
}

// Wind profile generator
double wind_speed(int t) {
    return 8.0 + 3.0 * sin(2 * M_PI * t / 60.0);  // varies between 5–11 m/s
}

// Battery storage
struct Battery {
    double soc;  // state of charge (0 to 1)
    double charge_eff = 0.95;
    double discharge_eff = 0.95;
    double capacity = 5.0;  // MWh

    double charge(double power, double dt) {
        double delta = power * dt / capacity;
        soc = min(1.0, soc + delta * charge_eff);
        return delta;
    }

    double discharge(double power, double dt) {
        double delta = power * dt / capacity;
        soc = max(0.0, soc - delta / discharge_eff);
        return -delta;
    }

    double get_energy() const {
        return soc * capacity;
    }
};

int main() {
    Battery battery{0.5};  // 50% initial charge
    vector<double> grid_output_log, wind_power_log, soc_log;

    for (int t = 0; t < SimTime; ++t) {
        // Predict future wind
        vector<double> wind_pred;
        for (int k = 0; k < Horizon; ++k)
            wind_pred.push_back(wind_speed(t + k));

        // Predict turbine output
        vector<double> power_pred;
        for (double w : wind_pred)
            power_pred.push_back(turbine_power_output(w));

        // Brute-force MPC: test discharge/charge strategies
        double best_cost = numeric_limits<double>::infinity();
        double best_grid = 0.0;

        for (double u = -0.5; u <= 0.5; u += 0.05) {  // battery power (MW), negative = charge
            double soc_temp = battery.soc;
            double cost = 0.0;

            for (int k = 0; k < Horizon; ++k) {
                double p_turbine = power_pred[k];
                double p_battery = u;

                soc_temp += p_battery * dt / battery.capacity;
                soc_temp = max(0.0, min(1.0, soc_temp));

                double p_grid = p_turbine + p_battery;
                cost += pow(p_grid - 0.7, 2);  // track 0.7 MW target
            }

            if (cost < best_cost) {
                best_cost = cost;
                best_grid = power_pred[0] + u;
            }
        }

        // Apply best control
        double wind_now = wind_speed(t);
        double p_turbine = turbine_power_output(wind_now);
        double u_apply = best_grid - p_turbine;

        if (u_apply >= 0) battery.discharge(u_apply, dt);
        else battery.charge(-u_apply, dt);

        double p_grid = p_turbine + u_apply;

        // Log
        grid_output_log.push_back(p_grid);
        wind_power_log.push_back(p_turbine);
        soc_log.push_back(battery.get_energy());
    }

    // Output
    cout << "Time,GridPower,WindPower,StorageMWh" << endl;
    for (int i = 0; i < SimTime; ++i) {
        cout << i << "," << grid_output_log[i] << "," << wind_power_log[i] << "," << soc_log[i] << endl;
    }

    return 0;
}
