#ifndef WIND_FARM_MODEL_H
#define WIND_FARM_MODEL_H
#include <Eigen/Dense>

class WindFarmModel {
private:
    // Parameters
    int n_turbines = 5; // Number of turbines
    double rho = 1.225; // Air density (kg/m^3)
    double A = 1256.0; // Rotor area (m^2, 20m radius)
    double E_max = 1000.0; // Battery capacity (kWh)
    double dt = 60.0; // Time step (s)
    double beta_max = 0.349; // Max pitch angle (20 deg)
    double P_batt_max = 200.0; // Max charge/discharge (kW)
    double eta_charge = 0.95; // Charging efficiency
    double eta_discharge = 0.95; // Discharging efficiency

public:
    Eigen::VectorXd simulate(const Eigen::VectorXd& x, const Eigen::VectorXd& u, double v_w);
    double calculate_power(double v_w, double beta);
    int get_state_size() const { return n_turbines + 1; } // States: beta_1,...,beta_n, SoC
    int get_control_size() const { return n_turbines + 2; } // Controls: beta_1,...,beta_n, P_charge, P_discharge
};

#endif
