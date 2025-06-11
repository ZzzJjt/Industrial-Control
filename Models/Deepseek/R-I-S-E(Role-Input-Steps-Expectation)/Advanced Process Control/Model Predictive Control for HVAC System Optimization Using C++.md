class HVACSystem {
public:
    double indoorTemperature;
    double indoorHumidity;
    double energyConsumption;

    HVACSystem(double initIndoorTemp, double initIndoorHumidity)
        : indoorTemperature(initIndoorTemp), indoorHumidity(initIndoorHumidity), energyConsumption(0.0) {}

    void update(double outdoorTemp, double occupancyLevel, double heatingCoolingRate) {
        // Simplified thermal dynamics
        indoorTemperature += heatingCoolingRate * (outdoorTemp - indoorTemperature);
        
        // Humidity control logic
        if(indoorTemperature > 25) { // Example condition
            indoorHumidity -= 0.5 * occupancyLevel; // Simplified humidity adjustment
        }
        
        // Energy consumption calculation
        energyConsumption += std::abs(heatingCoolingRate) * 0.1; // Simplified energy consumption
    }
};

class MPCController {
private:
    HVACSystem* hvacSystem;
    double predictionHorizon;
    double controlHorizon;

public:
    MPCController(HVACSystem* system, double predHorizon, double ctrlHorizon)
        : hvacSystem(system), predictionHorizon(predHorizon), controlHorizon(ctrlHorizon) {}

    std::vector<double> optimize(double outdoorTemp, double occupancyLevel) {
        // Simplified optimization process
        // In practice, use a library like IPOPT for solving the optimization problem
        
        std::vector<double> optimalControlActions(controlHorizon, 0.0);
        
        // Example: Optimize heating/cooling rate for the next hour
        double targetTemperature = 22.0; // Desired setpoint
        double heatingCoolingRate = (targetTemperature - hvacSystem->indoorTemperature) * 0.1;
        
        optimalControlActions[0] = heatingCoolingRate;
        
        return optimalControlActions;
    }
};

void applyConstraints(std::vector<double>& controlActions, double maxEnergyLimit) {
    double totalEnergyEstimate = 0.0;
    for(auto& action : controlActions) {
        totalEnergyEstimate += std::abs(action); // Simplified energy estimate
        if(totalEnergyEstimate > maxEnergyLimit) {
            action = 0.0; // Adjust control action to respect energy limit
        }
    }
}

void simulateScenarios() {
    HVACSystem hvacSys(20.0, 50.0); // Initial indoor temp and humidity
    MPCController mpcCtrl(&hvacSys, 24.0, 4.0); // Prediction and control horizons
    
    std::vector<std::pair<double, double>> scenarios = {{30.0, 10}, {5.0, 2}, /* ... */};
    
    for(const auto& scenario : scenarios) {
        double outdoorTemp = scenario.first;
        double occupancyLevel = scenario.second;
        
        auto controlActions = mpcCtrl.optimize(outdoorTemp, occupancyLevel);
        applyConstraints(controlActions, 1000.0); // Example energy limit
        
        for(auto action : controlActions) {
            hvacSys.update(outdoorTemp, occupancyLevel, action);
            // Output or log system state
            std::cout << "Indoor Temp: " << hvacSys.indoorTemperature << ", Energy Consumption: " << hvacSys.energyConsumption << "\n";
        }
    }
}
