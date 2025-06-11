// WindTurbine.h
#pragma once
#include <cmath>

class WindTurbine {
public:
    double rho;       // Air density
    double A;         // Swept area
    double Cp;        // Power coefficient
    double omega;     // Rotor speed (rad/s)
    double beta;      // Pitch angle (deg)
    double maxTorque;
    double power;

    WindTurbine(double airDensity, double sweptArea, double cp, double torqueLimit);

    void update(double windSpeed, double desiredTorque);
};

// WindTurbine.cpp
WindTurbine::WindTurbine(double airDensity, double sweptArea, double cp, double torqueLimit)
    : rho(airDensity), A(sweptArea), Cp(cp), maxTorque(torqueLimit), omega(0.0), beta(0.0), power(0.0) {}

void WindTurbine::update(double windSpeed, double desiredTorque) {
    if (desiredTorque > maxTorque) desiredTorque = maxTorque;

    // Simple MPPT: Assume optimal tip-speed ratio
    double lambda = 7.0; // Optimal tip-speed ratio
    omega = lambda * windSpeed / 50.0; // Simplified relation

    power = 0.5 * rho * A * Cp * pow(windSpeed, 3);
}

// BatteryStorage.h
#pragma once

class BatteryStorage {
public:
    double capacity;   // kWh
    double soc;        // State of Charge (0 to 1)
    double efficiency;
    double maxChargeRate;
    double maxDischargeRate;

    BatteryStorage(double cap, double eff, double chargeRate, double dischargeRate);

    void charge(double power, double dt);
    void discharge(double power, double dt);
};

// BatteryStorage.cpp
BatteryStorage::BatteryStorage(double cap, double eff, double chargeRate, double dischargeRate)
    : capacity(cap), soc(0.5), efficiency(eff), maxChargeRate(chargeRate), maxDischargeRate(dischargeRate) {}

void BatteryStorage::charge(double inputPower, double dt) {
    double energyIn = std::min(inputPower * dt * efficiency, maxChargeRate * dt);
    soc = std::min(soc + energyIn / capacity, 1.0);
}

void BatteryStorage::discharge(double requestedPower, double dt) {
    double energyOut = std::min(requestedPower * dt / efficiency, maxDischargeRate * dt);
    soc = std::max(soc - energyOut / capacity, 0.0);
}

#include <acado_optimal_control.hpp>
#include <acado_gnuplot.hpp>

USING_NAMESPACE_ACADO

int main() {
    DifferentialState   x_power, x_soc;
    Control             u_torque, u_battery;
    Parameter           t;
    OnlineData          wind_speed;

    Function f;
    f << dot(x_power) == 0.5 * 1.225 * 314 * 0.45 * pow(wind_speed, 3); // Power from wind
    f << dot(x_soc) == (-u_battery / 1000.0);                           // Battery SOC change

    DynamicOptimizationProblem opt(0.0, 10.0, 10); // 10s horizon, 10 steps

    opt.minimizeMayerTerm( (x_power - 200.0)*(x_power - 200.0) ); // Track 200 MW
    opt.minimizeLagrangeTerm( 0.1*u_torque*u_torque + 0.05*u_battery*u_battery );

    opt.subjectTo( f );
    opt.subjectTo( AT_START, x_power == 180.0 );
    opt.subjectTo( AT_START, x_soc == 0.6 );

    opt.subjectTo( x_soc >= 0.2 );
    opt.subjectTo( x_soc <= 0.9 );
    opt.subjectTo( x_power >= 150.0 );
    opt.subjectTo( x_power <= 250.0 );
    opt.subjectTo( u_torque >= -1e5 );
    opt.subjectTo( u_torque <= 1e5 );
    opt.subjectTo( u_battery >= -50.0 );
    opt.subjectTo( u_battery <= 50.0 );

    OptimizationAlgorithm algorithm(opt);
    algorithm.set(PRINTLEVEL, MEDIUM);
    algorithm.solve();

    return 0;
}

double computeDispatchCommand(double predictedWindPower, double gridDemand, double currentSOC) {
    double availablePower = predictedWindPower;

    // If grid demand exceeds available wind, discharge battery
    if (gridDemand > availablePower && currentSOC > 0.3) {
        return availablePower + (currentSOC - 0.3) * 50.0; // Discharge proportional to SOC
    }

    // If wind > demand, charge battery
    else if (availablePower > gridDemand && currentSOC < 0.8) {
        return gridDemand; // Keep excess for charging
    }

    // Otherwise match demand
    else {
        return min(gridDemand, availablePower);
    }
}

int main() {
    WindTurbine turbine(1.225, 314, 0.45, 1e5);
    BatteryStorage battery(1000, 0.9, 50, 50);

    double windProfile[] = {8, 9, 10, 12, 15, 13, 10, 8, 6, 5}; // m/s
    double gridDemand = 200.0; // MW
    double dt = 1.0; // Time step in seconds

    for (int i = 0; i < 10; ++i) {
        double windSpeed = windProfile[i];
        turbine.update(windSpeed, 0.0); // Assume controller adjusts torque

        double generatedPower = turbine.power;
        double dispatchPower = computeDispatchCommand(generatedPower, gridDemand, battery.soc);

        double excess = generatedPower - dispatchPower;
        if (excess > 0) {
            battery.charge(excess, dt);
        } else {
            battery.discharge(-excess, dt);
        }

        std::cout << "Time: " << i << " | Wind: " << windSpeed
                  << " | Gen: " << generatedPower
                  << " | Dispatch: " << dispatchPower
                  << " | SOC: " << battery.soc << std::endl;
    }

    return 0;
}
