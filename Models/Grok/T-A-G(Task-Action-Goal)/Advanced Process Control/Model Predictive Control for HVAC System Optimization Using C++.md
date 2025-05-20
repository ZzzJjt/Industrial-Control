#ifndef HVAC_MODEL_H
#define HVAC_MODEL_H
#include <Eigen/Dense>

class HVACModel {
private:
    // Parameters
    double m = 10000.0; // Thermal mass (kg)
    double cp = 1.0; // Specific heat (kJ/kg°C)
    double V = 500.0; // Volume (m³)
    double rho = 1.2; // Air density (kg/m³)
    double k_loss = 0.1; // Heat loss coefficient (kW/°C)
    double m_loss = 0.001; // Moisture loss coefficient (kg/s per kg/m³)
    double Q_occ = 0.1; // Heat per occupant (kW)
    double M_occ = 1e-5; // Moisture per occupant (kg/s)
    double dt = 60.0; // Time step (s)

public:
    Eigen::Vector2d simulate(double T, double H, double Q_HVAC, double M_HVAC, double T_ext, double H_ext, int occ);
};

#endif
