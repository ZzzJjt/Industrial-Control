```
import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl

# Create the antecedent (input) and consequent (output) objects
temperature_error = ctrl.Antecedent(np.arange(-50, 50, 1), 'temperature_error')
heater_output = ctrl.Consequent(np.arange(0, 100, 1), 'heater_output')

# Auto-membership function population is possible with .automf(3, 5, or 7)
temperature_error.automf(3)

# Custom membership functions can be built interactively with a familiar,
# Pythonic API
heater_output['low'] = fuzz.trimf(heater_output.universe, [0, 0, 50])
heater_output['medium'] = fuzz.trimf(heater_output.universe, [0, 50, 100])
heater_output['high'] = fuzz.trimf(heater_output.universe, [50, 100, 100])

# Define rules based on our fuzzy logic system
rule1 = ctrl.Rule(temperature_error['bad'], heater_output['high'])
rule2 = ctrl.Rule(temperature_error['good'], heater_output['low'])
rule3 = ctrl.Rule(temperature_error['average'], heater_output['medium'])

# Create a control system
heating_ctrl = ctrl.ControlSystem([rule1, rule2, rule3])

# Create a controller object based on the system
heating = ctrl.ControlSystemSimulation(heating_ctrl)
def simulate_reactor(initial_temp, target_temp, steps=100):
    current_temp = initial_temp
    temperatures = []
    for _ in range(steps):
        # Calculate the error
        error = target_temp - current_temp
        heating.temperature_error = error
        
        # Crunch the numbers
        heating.compute()
        
        # Apply the output of the fuzzy controller to adjust the temperature
        current_temp += heating.output['heater_output'] / 100.0 - (target_temp - current_temp) * 0.01
        temperatures.append(current_temp)
    
    return temperatures

# Example usage
temperatures = simulate_reactor(initial_temp=200, target_temp=250, steps=100)
print(temperatures)
import matplotlib.pyplot as plt

plt.figure()
plt.plot(range(100), temperatures, 'b', label='Temperature')
plt.xlabel('Time (steps)')
plt.ylabel('Temperature')
plt.title('Fuzzy Logic Controlled Temperature Over Time')
plt.legend(loc='best')
plt.grid(True)
plt.show()
```
