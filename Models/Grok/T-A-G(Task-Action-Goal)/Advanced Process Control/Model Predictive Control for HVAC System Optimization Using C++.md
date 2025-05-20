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

#include "hvac_model.h"

Eigen::Vector2d HVACModel::simulate(double T, double H, double Q_HVAC, double M_HVAC, double T_ext, double H_ext, int occ) {
    // State: [T, H]
    // Inputs: Q_HVAC (kW), M_HVAC (kg/s)
    // External: T_ext (°C), H_ext (kg/m³), occ (people)

    // Thermal dynamics: dT/dt = (Q_HVAC - k_loss*(T - T_ext) + Q_occ*occ) / (m*cp)
    double Q_loss = k_loss * (T - T_ext);
    double dT_dt = (Q_HVAC - Q_loss + Q_occ * occ) / (m * cp);
    
    // Humidity dynamics: dH/dt = (M_HVAC - m_loss*(H - H_ext) + M_occ*occ) / (V*rho)
    double M_loss = m_loss * (H - H_ext);
    double dH_dt = (M_HVAC - M_loss + M_occ * occ) / (V * rho);
    
    // Update states
    double T_next = T + dT_dt * dt;
    double H_next = H + dH_dt * dt;
    
    return Eigen::Vector2d(T_next, H_next);
}
