#include <iostream>
#include <vector>
#include <cmath>
#include <Eigen/Dense>
#include <memory>
#include <algorithm>
#include <random>

using namespace Eigen;

class WindTurbine {
private:
    double rotorDiameter;    // m
    double ratedPower;       // kW
    double cutInSpeed;       // m/s
    double ratedSpeed;       // m/s
    double cutOutSpeed;      // m/s
    double generatorEfficiency;
    double currentSpeed;     // rad/s
    double bladePitch;       // degrees
    double powerOutput;      // kW

public:
    WindTurbine(double diameter, double power, double minSpeed, double optSpeed, double maxSpeed)
        : rotorDiameter(diameter), ratedPower(power), cutInSpeed(minSpeed),
          ratedSpeed(optSpeed), cutOutSpeed(maxSpeed), generatorEfficiency(0.95),
          currentSpeed(0.0), bladePitch(0.0), powerOutput(0.0) {}

    void update(double windSpeed, double pitchControl, double dt) {
        // Apply pitch control constraints
        bladePitch = std::clamp(bladePitch + pitchControl, 0.0, 25.0);
        
        // Calculate power coefficient (Cp) based on pitch and tip-speed ratio
        double tipSpeedRatio = (currentSpeed * rotorDiameter/2) / std::max(windSpeed, 0.1);
        double cp = calculatePowerCoefficient(tipSpeedRatio, bladePitch);
        
        // Mechanical power from wind
        double airDensity = 1.225; // kg/m³
        double sweptArea = M_PI * pow(rotorDiameter/2, 2);
        double mechanicalPower = 0.5 * cp * airDensity * sweptArea * pow(windSpeed, 3);
        
        // Generator power output
        powerOutput = std::min(mechanicalPower * generatorEfficiency, ratedPower);
        
        // Only produce power if within operational limits
        if (windSpeed < cutInSpeed || windSpeed > cutOutSpeed) {
            powerOutput = 0.0;
        }
        
        // Simple rotor dynamics
        double torque = mechanicalPower / std::max(currentSpeed, 0.1);
        double inertia = 1000.0; // kg·m²
        currentSpeed += (torque - currentSpeed*0.1) / inertia * dt; // Simple friction model
    }

    double calculatePowerCoefficient(double lambda, double beta) {
        // Simplified Cp-lambda curve model
        beta = beta * M_PI / 180.0; // Convert to radians
        double cp_max = 0.48;
        double lambda_opt = 8.0;
        
        // Reduce Cp based on pitch angle
        double cp = cp_max * (1 - 0.035 * pow(beta, 1.5));
        
        // Reduce Cp based on tip-speed ratio deviation
        cp *= exp(-0.5 * pow((lambda - lambda_opt)/(0.1*lambda_opt), 2));
        
        return std::max(cp, 0.0);
    }

    double getPowerOutput() const { return powerOutput; }
    double getBladePitch() const { return bladePitch; }
    void setBladePitch(double pitch) { bladePitch = pitch; }
};

class EnergyStorage {
private:
    double capacity;         // kWh
    double currentEnergy;    // kWh
    double maxChargeRate;    // kW
    double maxDischargeRate; // kW
    double efficiency;

public:
    EnergyStorage(double cap, double chargeRate, double dischargeRate)
        : capacity(cap), currentEnergy(cap/2), maxChargeRate(chargeRate),
          maxDischargeRate(dischargeRate), efficiency(0.92) {}

    void update(double power, double dt) {
        // Charge (positive power) or discharge (negative power)
        double deltaEnergy = 0.0;
        if (power > 0) {
            double chargePower = std::min(power, maxChargeRate);
            deltaEnergy = chargePower * dt / 3600.0 * efficiency;
        } else {
            double dischargePower = std::max(power, -maxDischargeRate);
            deltaEnergy = dischargePower * dt / 3600.0 / efficiency;
        }
        
        currentEnergy = std::clamp(currentEnergy + deltaEnergy, 0.0, capacity);
    }

    double getAvailableEnergy() const { return currentEnergy; }
    double getMaxDischargeRate() const { return maxDischargeRate; }
};

class WindFarmMPC {
private:
    std::vector<WindTurbine> turbines;
    EnergyStorage storage;
    double totalPowerOutput;
    double gridDemand;
    double predictionHorizon;
    double timeStep;
    MatrixXd stateTransitionMatrix;
    MatrixXd controlMatrix;
    MatrixXd costMatrixQ;
    MatrixXd costMatrixR;

    // Wind forecast
    std::vector<double> windForecast;
    std::default_random_engine generator;
    std::normal_distribution<double> windNoise;

public:
    WindFarmMPC(int numTurbines, double storageCapacity)
        : turbines(), storage(storageCapacity, storageCapacity/4, storageCapacity/4),
          totalPowerOutput(0.0), gridDemand(0.0), predictionHorizon(10), timeStep(1.0),
          generator(std::random_device{}()), windNoise(0.0, 0.5) {
        
        // Initialize turbines
        for (int i = 0; i < numTurbines; ++i) {
            turbines.emplace_back(80.0, 2000.0, 3.0, 12.0, 25.0);
        }
        
        // Initialize MPC matrices (simplified model)
        int numStates = numTurbines + 1; // Turbine powers + storage
        int numControls = numTurbines + 1; // Pitch controls + storage power
        
        stateTransitionMatrix = MatrixXd::Identity(numStates, numStates);
        controlMatrix = MatrixXd::Zero(numStates, numControls);
        costMatrixQ = MatrixXd::Identity(numStates, numStates);
        costMatrixR = MatrixXd::Identity(numControls, numControls);
        
        // Generate wind forecast
        updateWindForecast();
    }

    void updateWindForecast() {
        windForecast.clear();
        double baseSpeed = 10.0 + 5.0 * sin(timeStep * 0.1);
        
        for (int i = 0; i < predictionHorizon; ++i) {
            double speed = baseSpeed + 2.0 * sin(timeStep * 0.2 + i * 0.5) + windNoise(generator);
            windForecast.push_back(std::max(speed, 0.0));
        }
    }

    VectorXd predictPowerOutput(const VectorXd& pitchAngles) {
        VectorXd prediction(turbines.size());
        
        for (size_t i = 0; i < turbines.size(); ++i) {
            // Simplified prediction - in reality would use full dynamics
            double cp = turbines[i].calculatePowerCoefficient(8.0, pitchAngles(i));
            double airDensity = 1.225;
            double sweptArea = M_PI * pow(turbines[i].getRotorDiameter()/2, 2);
            prediction(i) = 0.5 * cp * airDensity * sweptArea * pow(windForecast[0], 3) * 0.95;
        }
        
        return prediction;
    }

    double mpcCostFunction(const VectorXd& u) {
        // u contains pitch angles for all turbines + storage power
        double cost = 0.0;
        
        // Predict power output for current control
        VectorXd pitchAngles = u.head(turbines.size());
        VectorXd powerPred = predictPowerOutput(pitchAngles);
        double totalPower = powerPred.sum() + u(turbines.size());
        
        // Power tracking cost
        cost += 10.0 * pow(totalPower - gridDemand, 2);
        
        // Turbine load balancing cost
        double meanPower = powerPred.mean();
        for (int i = 0; i < powerPred.size(); ++i) {
            cost += 2.0 * pow(powerPred(i) - meanPower, 2);
        }
        
        // Control effort cost (pitch changes and storage usage)
        for (int i = 0; i < u.size(); ++i) {
            cost += 0.1 * pow(u(i), 2);
        }
        
        // Storage state cost (encourage keeping some reserve)
        double storageUse = u(turbines.size());
        double storageLevel = storage.getAvailableEnergy();
        cost += 0.5 * pow(storageLevel - storage.getCapacity()/2, 2);
        
        return cost;
    }

    void updateGridDemand(double demand) {
        gridDemand = demand;
    }

    void controlStep() {
        // Update wind forecast
        updateWindForecast();
        
        // Current state (turbine powers + storage level)
        VectorXd currentState(turbines.size() + 1);
        for (size_t i = 0; i < turbines.size(); ++i) {
            currentState(i) = turbines[i].getPowerOutput();
        }
        currentState(turbines.size()) = storage.getAvailableEnergy();
        
        // Initial guess for controls (current pitch angles + zero storage power)
        VectorXd u0(turbines.size() + 1);
        for (size_t i = 0; i < turbines.size(); ++i) {
            u0(i) = turbines[i].getBladePitch();
        }
        u0(turbines.size()) = 0.0;
        
        // Solve MPC problem (simplified - in practice would use proper optimization)
        VectorXd u = u0;
        double minCost = mpcCostFunction(u0);
        
        // Simple gradient-based search (replace with proper QP solver in real implementation)
        for (int iter = 0; iter < 20; ++iter) {
            VectorXd trial = u0;
            for (int i = 0; i < trial.size(); ++i) {
                trial(i) += 0.1 * (rand() % 100 - 50) / 50.0;
            }
            
            double cost = mpcCostFunction(trial);
            if (cost < minCost) {
                minCost = cost;
                u = trial;
            }
        }
        
        // Apply controls
        for (size_t i = 0; i < turbines.size(); ++i) {
            double pitchChange = u(i) - turbines[i].getBladePitch();
            turbines[i].update(windForecast[0], pitchChange, timeStep);
        }
        
        // Update storage
        storage.update(u(turbines.size()), timeStep);
        
        // Update total power output
        totalPowerOutput = 0.0;
        for (const auto& turbine : turbines) {
            totalPowerOutput += turbine.getPowerOutput();
        }
        totalPowerOutput += std::min(0.0, u(turbines.size())); // Add storage discharge
    }

    double getTotalPowerOutput() const { return totalPowerOutput; }
    const std::vector<WindTurbine>& getTurbines() const { return turbines; }
    const EnergyStorage& getStorage() const { return storage; }
};

int main() {
    // Create wind farm with 5 turbines and 10 MWh storage
    WindFarmMPC windFarm(5, 10000.0);
    
    // Simulation parameters
    const int totalSteps = 100;
    std::vector<double> powerOutput;
    std::vector<double> gridDemand;
    std::vector<std::vector<double>> turbinePowers;
    
    // Run simulation
    for (int step = 0; step < totalSteps; ++step) {
        // Update grid demand (simulated)
        double demand = 6000.0 + 2000.0 * sin(step * 0.1);
        windFarm.updateGridDemand(demand);
        
        // Execute MPC control step
        windFarm.controlStep();
        
        // Record data
        powerOutput.push_back(windFarm.getTotalPowerOutput());
        gridDemand.push_back(demand);
        
        std::vector<double> turbinePower;
        for (const auto& t : windFarm.getTurbines()) {
            turbinePower.push_back(t.getPowerOutput());
        }
        turbinePowers.push_back(turbinePower);
        
        // Print status
        std::cout << "Step " << step << ": Demand = " << demand 
                  << " kW, Output = " << windFarm.getTotalPowerOutput() 
                  << " kW, Storage = " << windFarm.getStorage().getAvailableEnergy() 
                  << " kWh" << std::endl;
    }
    
    // Plot results (would use a plotting library in real implementation)
    // ... plotting code would go here ...
    
    return 0;
}
