#include <iostream>
#include <vector>
#include <Eigen/Dense>
#include <cmath>
#include <random>

using namespace std;
using namespace Eigen;

const double Ts = 1.0;          // Time step (minutes)
const int Horizon = 10;         // MPC prediction horizon
const double Tset = 22.0;       // Target indoor temperature (°C)
const double Tmin = 18.0;
const double Tmax = 26.0;
const double PowerLimit = 5.0;  // Max heating/cooling power (kW)
const double RoomThermalMass = 5.0;
const double ThermalLoss = 0.1; // Heat exchange with outside

// Simple dynamics: next_temp = current_temp + Ts * (heating_power - thermal_loss * (current_temp - outside_temp)) / thermal_mass
double nextTemperature(double T_current, double power, double T_out) {
    return T_current + Ts * (power - ThermalLoss * (T_current - T_out)) / RoomThermalMass;
}

// Generate random external temperature (simulating day/night)
double externalTemperature(int t_minute) {
    return 10.0 + 10.0 * sin(2 * M_PI * t_minute / 1440.0); // 24h sine wave
}

// Generate occupancy level (0 or 1)
int occupancy(int t_minute) {
    return (t_minute % 1440 >= 8 * 60 && t_minute % 1440 <= 18 * 60) ? 1 : 0;
}

int main() {
    const int SimTime = 240; // total time (minutes)
    double T_in = 20.0; // initial indoor temperature
    vector<double> T_record, Power_record;

    for (int t = 0; t < SimTime; ++t) {
        double T_out = externalTemperature(t);
        int occ = occupancy(t);

        // Construct prediction
        VectorXd T_pred(Horizon + 1);
        VectorXd U(Horizon);
        T_pred(0) = T_in;
        double T_temp = T_in;

        // Simple MPC: try constant power across horizon and pick best
        double best_cost = 1e9;
        double best_u = 0.0;

        for (double u_test = -PowerLimit; u_test <= PowerLimit; u_test += 0.1) {
            double cost = 0.0;
            T_temp = T_in;
            for (int k = 0; k < Horizon; ++k) {
                T_temp = nextTemperature(T_temp, u_test, T_out);
                double comfort_weight = occ ? 2.0 : 0.5;
                cost += comfort_weight * pow(T_temp - Tset, 2) + 0.1 * pow(u_test, 2);
            }
            if (cost < best_cost) {
                best_cost = cost;
                best_u = u_test;
            }
        }

        // Apply best_u
        T_in = nextTemperature(T_in, best_u, T_out);
        T_record.push_back(T_in);
        Power_record.push_back(best_u);
    }

    // Output results
    cout << "Time(min),IndoorTemp(C),Power(kW)" << endl;
    for (int i = 0; i < SimTime; ++i) {
        cout << i << "," << T_record[i] << "," << Power_record[i] << endl;
    }

    return 0;
}
