#include <iostream>
#include <vector>
#include <cmath>
#include <memory>
#include <Eigen/Dense>  // Requires Eigen library for matrix operations

using namespace Eigen;

class HVACSystem {
private:
    // System parameters
    double thermalMass;          // Building thermal mass (J/°C)
    double thermalResistance;    // Thermal resistance (°C/W)
    double humidityCapacity;     // Air humidity capacity (kg/%RH)
    double maxCoolingPower;      // Maximum cooling power (W)
    double maxHeatingPower;      // Maximum heating power (W)
    double maxHumidification;    // Maximum humidification rate (%RH/s)
    double maxDehumidification;  // Maximum dehumidification rate (%RH/s)
    
    // State variables
    double indoorTemp;           // Current indoor temperature (°C)
    double indoorHumidity;       // Current indoor humidity (%RH)
    
public:
    HVACSystem(double tempInit = 22.0, double humidityInit = 50.0) 
        : thermalMass(1.5e6), thermalResistance(0.01), humidityCapacity(1.2),
          maxCoolingPower(5000), maxHeatingPower(4000),
          maxHumidification(0.1), maxDehumidification(0.15),
          indoorTemp(tempInit), indoorHumidity(humidityInit) {}
    
    // Update system state based on control inputs and external conditions
    void update(double heatingPower, double coolingPower, 
                double humidification, double dehumidification,
                double outdoorTemp, double outdoorHumidity, 
                int occupancy, double dt) {
        
        // Clamp control inputs to valid ranges
        heatingPower = std::clamp(heatingPower, 0.0, maxHeatingPower);
        coolingPower = std::clamp(coolingPower, 0.0, maxCoolingPower);
        humidification = std::clamp(humidification, 0.0, maxHumidification);
        dehumidification = std::clamp(dehumidification, 0.0, maxDehumidification);
        
        // Temperature dynamics
        double tempDifference = outdoorTemp - indoorTemp;
        double heatTransfer = tempDifference / thermalResistance;
        double netHeat = heatingPower - coolingPower + heatTransfer;
        double occupancyHeat = occupancy * 100; // 100W per person
        indoorTemp += (netHeat + occupancyHeat) / thermalMass * dt;
        
        // Humidity dynamics
        double humidityDifference = outdoorHumidity - indoorHumidity;
        double humidityTransfer = humidityDifference * 0.05; // Simplified model
        double netHumidity = humidification - dehumidification + humidityTransfer;
        double occupancyHumidity = occupancy * 0.01; // 0.01%RH per person
        indoorHumidity += (netHumidity + occupancyHumidity) / humidityCapacity * dt;
        
        // Ensure values stay within reasonable bounds
        indoorTemp = std::clamp(indoorTemp, 10.0, 35.0);
        indoorHumidity = std::clamp(indoorHumidity, 20.0, 80.0);
    }
    
    // Get current state
    std::pair<double, double> getState() const {
        return {indoorTemp, indoorHumidity};
    }
};

class MPController {
private:
    int predictionHorizon;
    double timeStep;
    MatrixXd A;  // State transition matrix
    MatrixXd B;  // Control matrix
    MatrixXd Q;  // State cost matrix
    MatrixXd R;  // Control cost matrix
    
    // Comfort range
    double tempSetpoint;
    double humiditySetpoint;
    double tempTolerance;
    double humidityTolerance;
    
public:
    MPController(int horizon = 10, double dt = 1.0) 
        : predictionHorizon(horizon), timeStep(dt),
          tempSetpoint(22.0), humiditySetpoint(50.0),
          tempTolerance(1.5), humidityTolerance(5.0) {
        
        // Initialize matrices (simplified linear model)
        A = MatrixXd::Identity(2, 2);
        A(0,0) = 0.95;  // Temperature decay factor
        A(1,1) = 0.97;  // Humidity decay factor
        
        B = MatrixXd::Zero(2, 4);
        B(0,0) = 0.002;  // Heating effect on temp
        B(0,1) = -0.002; // Cooling effect on temp
        B(1,2) = 0.03;   // Humidification effect
        B(1,3) = -0.04;  // Dehumidification effect
        
        // Cost matrices - balance between comfort and energy use
        Q = MatrixXd::Identity(2, 2);
        Q(0,0) = 10.0;  // Temperature tracking weight
        Q(1,1) = 5.0;   // Humidity tracking weight
        
        R = MatrixXd::Identity(4, 4);
        R(0,0) = 0.01;  // Heating cost
        R(1,1) = 0.01;  // Cooling cost
        R(2,2) = 0.05;  // Humidification cost
        R(3,3) = 0.05;  // Dehumidification cost
    }
    
    // Solve MPC problem to get optimal control inputs
    Vector4d solveMPC(const Vector2d& currentState, 
                     const Vector2d& disturbance, 
                     int occupancy) const {
        
        Vector2d target(tempSetpoint, humiditySetpoint);
        
        // Adjust setpoint based on occupancy
        if (occupancy > 0) {
            target(0) += occupancy * 0.1;  // Slightly higher temp with more people
        }
        
        // Simple MPC implementation (in practice would use a proper QP solver)
        // This is a simplified version for demonstration
        
        Vector4d controlInputs = Vector4d::Zero();
        
        // Calculate errors
        Vector2d error = currentState - target;
        
        // Simple proportional control based on MPC principles
        double tempError = error(0);
        double humidityError = error(1);
        
        // Heating/cooling decision
        if (tempError > tempTolerance) {
            controlInputs(1) = std::min(0.5 * tempError, 1.0);  // Cooling
        } else if (tempError < -tempTolerance) {
            controlInputs(0) = std::min(-0.5 * tempError, 1.0);  // Heating
        }
        
        // Humidification/dehumidification decision
        if (humidityError > humidityTolerance) {
            controlInputs(3) = std::min(0.3 * humidityError, 1.0);  // Dehumidify
        } else if (humidityError < -humidityTolerance) {
            controlInputs(2) = std::min(-0.2 * humidityError, 1.0);  // Humidify
        }
        
        // Adjust for occupancy
        if (occupancy > 0) {
            // Increase ventilation with more people
            controlInputs(1) = std::min(controlInputs(1) + occupancy * 0.05, 1.0);
            controlInputs(3) = std::min(controlInputs(3) + occupancy * 0.03, 1.0);
        }
        
        return controlInputs;
    }
    
    // Update setpoints based on time of day or other factors
    void updateSetpoints(double temp, double humidity) {
        tempSetpoint = temp;
        humiditySetpoint = humidity;
    }
};

// Weather forecast simulator
class WeatherSimulator {
public:
    static std::pair<double, double> getForecast(int hour) {
        // Simple periodic weather pattern
        double temp = 15.0 + 10.0 * std::sin(hour * 3.14159 / 12.0);
        double humidity = 40.0 + 30.0 * std::sin(hour * 3.14159 / 24.0 + 0.5);
        return {temp, humidity};
    }
};

// Occupancy simulator
class OccupancySimulator {
public:
    static int predictOccupancy(int hour) {
        // Simple office building occupancy pattern
        if (hour >= 9 && hour < 17) {  // Work hours
            if (hour == 12) return 15;  // Lunch time
            return 10 + 5 * std::sin((hour - 13) * 3.14159 / 8.0);
        }
        return 0;  // After hours
    }
};

int main() {
    // Initialize system and controller
    HVACSystem hvac;
    MPController mpc;
    
    // Simulation parameters
    const int totalHours = 24;
    const double dt = 1.0 / 60.0;  // 1-minute time steps
    const int stepsPerHour = static_cast<int>(1.0 / dt);
    
    // Data logging
    std::vector<double> tempHistory;
    std::vector<double> humidityHistory;
    std::vector<double> energyUsage;
    
    // Simulation loop
    for (int hour = 0; hour < totalHours; ++hour) {
        for (int step = 0; step < stepsPerHour; ++step) {
            // Get current state
            auto [currentTemp, currentHumidity] = hvac.getState();
            
            // Get forecast and occupancy prediction
            auto [outdoorTemp, outdoorHumidity] = WeatherSimulator::getForecast(hour);
            int occupancy = OccupancySimulator::predictOccupancy(hour);
            
            // Solve MPC problem
            Vector2d currentState(currentTemp, currentHumidity);
            Vector2d disturbance(outdoorTemp, outdoorHumidity);
            Vector4d controls = mpc.solveMPC(currentState, disturbance, occupancy);
            
            // Scale controls to actual system limits
            double heating = controls(0) * hvac.getMaxHeatingPower();
            double cooling = controls(1) * hvac.getMaxCoolingPower();
            double humidify = controls(2) * hvac.getMaxHumidification();
            double dehumidify = controls(3) * hvac.getMaxDehumidification();
            
            // Update system
            hvac.update(heating, cooling, humidify, dehumidify,
                       outdoorTemp, outdoorHumidity, occupancy, dt);
            
            // Log data (once per hour)
            if (step == 0) {
                tempHistory.push_back(currentTemp);
                humidityHistory.push_back(currentHumidity);
                energyUsage.push_back(heating + cooling + humidify + dehumidify);
            }
        }
    }
    
    // Print results
    std::cout << "Hour\tTemp(°C)\tHumidity(%RH)\tEnergy(kWh)\n";
    for (int hour = 0; hour < totalHours; ++hour) {
        std::cout << hour << "\t" << std::fixed << std::setprecision(1) 
                  << tempHistory[hour] << "\t\t" << humidityHistory[hour] 
                  << "\t\t" << std::setprecision(2) << energyUsage[hour]/1000.0 << "\n";
    }
    
    return 0;
}
