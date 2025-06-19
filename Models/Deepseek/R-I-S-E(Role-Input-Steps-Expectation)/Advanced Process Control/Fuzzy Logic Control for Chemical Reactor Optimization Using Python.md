import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl

# Define fuzzy variables with their universe of discourse
temperature = ctrl.Antecedent(np.arange(200, 501, 1), 'temperature')
pressure = ctrl.Antecedent(np.arange(1, 10.1, 0.1), 'pressure')
reaction_rate = ctrl.Consequent(np.arange(0, 1.01, 0.01), 'reaction_rate')

# Temperature membership functions
temperature['low'] = fuzz.trimf(temperature.universe, [200, 200, 300])
temperature['medium'] = fuzz.trimf(temperature.universe, [250, 350, 450])
temperature['high'] = fuzz.trimf(temperature.universe, [400, 500, 500])

# Pressure membership functions
pressure['low'] = fuzz.trimf(pressure.universe, [1, 1, 4])
pressure['medium'] = fuzz.trimf(pressure.universe, [2, 5, 8])
pressure['high'] = fuzz.trimf(pressure.universe, [6, 10, 10])

# Reaction rate membership functions
reaction_rate['slow'] = fuzz.trimf(reaction_rate.universe, [0, 0, 0.4])
reaction_rate['moderate'] = fuzz.trimf(reaction_rate.universe, [0.2, 0.5, 0.8])
reaction_rate['fast'] = fuzz.trimf(reaction_rate.universe, [0.6, 1, 1])

rule1 = ctrl.Rule(temperature['low'] & pressure['low'], reaction_rate['slow'])
rule2 = ctrl.Rule(temperature['medium'] & pressure['medium'], reaction_rate['moderate'])
rule3 = ctrl.Rule(temperature['high'] & pressure['high'], reaction_rate['fast'])
rule4 = ctrl.Rule(temperature['high'] & pressure['low'], reaction_rate['moderate'])
rule5 = ctrl.Rule(temperature['low'] & pressure['high'], reaction_rate['moderate'])

# Create control system
reaction_ctrl = ctrl.ControlSystem([rule1, rule2, rule3, rule4, rule5])
reactor_sim = ctrl.ControlSystemSimulation(reaction_ctrl)

def run_fuzzy_controller(setpoint_temp=350, setpoint_pressure=5):
    temps, pressures, rates = [], [], []
    temp_error_integral = 0
    prev_temp_error = 0
    
    # Initial values
    current_temp = 300
    current_pressure = 4.0
    
    for t in range(100):
        # Apply fuzzy controller
        reactor_sim.input['temperature'] = current_temp
        reactor_sim.input['pressure'] = current_pressure
        reactor_sim.compute()
        
        # Get predicted reaction rate
        pred_rate = reactor_sim.output['reaction_rate']
        
        # Adjust coolant flow or heating based on error
        temp_error = setpoint_temp - current_temp
        temp_error_integral += temp_error
        derivative = temp_error - prev_temp_error
        
        # Simple PI adjustment for temperature
        delta_coolant = 0.5 * temp_error + 0.05 * temp_error_integral
        current_temp += -delta_coolant + np.random.normal(0, 0.5)
        current_pressure = 4 + np.sin(t/10)  # Simulated fluctuating pressure
        
        temps.append(current_temp)
        pressures.append(current_pressure)
        rates.append(pred_rate)
        
        prev_temp_error = temp_error
        
    return temps, pressures, rates
