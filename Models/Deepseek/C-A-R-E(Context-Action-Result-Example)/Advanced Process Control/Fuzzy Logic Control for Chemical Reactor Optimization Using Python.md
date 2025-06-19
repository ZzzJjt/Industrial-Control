To address the challenge of controlling a chemical reactor under uncertain and dynamically changing conditions, we will develop a **fuzzy logic control system** in Python. This system will be designed to regulate key reactor variables such as temperature and pressure. We'll utilize the `skfuzzy` library for implementing fuzzy logic rules and membership functions.

### Prerequisites

Before proceeding, ensure you have installed the necessary libraries:

```bash
pip install numpy scikit-fuzzy matplotlib
```

### Step 1: Define the Problem and System Parameters

We consider a simplified scenario where the goal is to maintain the reactor's temperature and pressure within desired ranges despite external disturbances.

### Step 2: Implement Fuzzy Logic Control System

Below is a complete Python script that demonstrates how to implement and simulate a fuzzy logic controller for a chemical reactor:

```python
import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt

# Step 1: Define universe variables and membership functions
temperature = ctrl.Antecedent(np.arange(250, 401, 1), 'temperature')  # Temperature range from 250K to 400K
pressure = ctrl.Antecedent(np.arange(0, 101, 1), 'pressure')  # Pressure range from 0 to 100 bar
control_action = ctrl.Consequent(np.arange(0, 101, 1), 'control_action')  # Control action output

# Membership functions for temperature
temperature['cold'] = fuzz.trimf(temperature.universe, [250, 250, 300])
temperature['normal'] = fuzz.trimf(temperature.universe, [280, 325, 370])
temperature['hot'] = fuzz.trimf(temperature.universe, [350, 400, 400])

# Membership functions for pressure
pressure['low'] = fuzz.trimf(pressure.universe, [0, 0, 50])
pressure['medium'] = fuzz.trimf(pressure.universe, [20, 50, 80])
pressure['high'] = fuzz.trimf(pressure.universe, [50, 100, 100])

# Membership functions for control action
control_action['decrease'] = fuzz.trimf(control_action.universe, [0, 0, 50])
control_action['maintain'] = fuzz.trimf(control_action.universe, [20, 50, 80])
control_action['increase'] = fuzz.trimf(control_action.universe, [50, 100, 100])

# Step 2: Define fuzzy rules
rule1 = ctrl.Rule(temperature['cold'] & pressure['low'], control_action['increase'])
rule2 = ctrl.Rule(temperature['normal'] | pressure['medium'], control_action['maintain'])
rule3 = ctrl.Rule(temperature['hot'] & pressure['high'], control_action['decrease'])

# Step 3: Create control system
control_system = ctrl.ControlSystem([rule1, rule2, rule3])
controller = ctrl.ControlSystemSimulation(control_system)

# Step 4: Simulate and visualize the response
temp_input = np.linspace(250, 400, 100)
press_input = np.linspace(0, 100, 100)
output_control = np.zeros((len(temp_input), len(press_input)))

for i, temp in enumerate(temp_input):
    for j, press in enumerate(press_input):
        controller.input['temperature'] = temp
        controller.input['pressure'] = press
        controller.compute()
        output_control[i, j] = controller.output['control_action']

# Plotting
X, Y = np.meshgrid(temp_input, press_input)
fig = plt.figure(figsize=(8, 6))
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(X, Y, output_control.T, cmap='viridis')

ax.set_xlabel('Temperature (K)')
ax.set_ylabel('Pressure (bar)')
ax.set_zlabel('Control Action')
plt.title("Fuzzy Logic Control Surface")
plt.show()

# Example: Handling sudden changes
controller.input['temperature'] = 360  # Sudden increase in temperature
controller.input['pressure'] = 85      # High pressure
controller.compute()
print(f"Recommended Control Action: {controller.output['control_action']}%")
```

### Explanation

- **Step 1:** Defines the input variables (`temperature`, `pressure`) and the output variable (`control_action`). Membership functions are created to represent linguistic terms like "cold", "normal", "hot" for temperature, and "low", "medium", "high" for pressure.
  
- **Step 2:** Establishes fuzzy rules that dictate how the control action should respond based on the current state of temperature and pressure.
  
- **Step 3:** Constructs a fuzzy control system using these rules, allowing for computation of the appropriate control action given specific inputs.
  
- **Step 4:** Demonstrates a simulation where the control surface is visualized, showing how different combinations of temperature and pressure influence the recommended control action. Additionally, an example is provided where the system handles a sudden change in temperature and pressure, outputting a recommended control action.

This fuzzy logic approach provides a flexible method for managing complex, nonlinear systems by allowing smooth transitions between states rather than abrupt changes, which could lead to instability or inefficiency in traditional control schemes.
