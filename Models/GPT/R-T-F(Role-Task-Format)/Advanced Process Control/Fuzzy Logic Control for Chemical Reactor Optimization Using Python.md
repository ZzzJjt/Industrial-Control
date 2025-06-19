import numpy as np
import matplotlib.pyplot as plt
import skfuzzy as fuzz
from skfuzzy import control as ctrl

# Define fuzzy variables
temperature = ctrl.Antecedent(np.arange(0, 101, 1), 'temperature')
pressure = ctrl.Antecedent(np.arange(0, 11, 1), 'pressure')
reaction_rate = ctrl.Consequent(np.arange(0, 101, 1), 'reaction_rate')

# Membership functions
temperature['low'] = fuzz.trimf(temperature.universe, [0, 0, 50])
temperature['medium'] = fuzz.trimf(temperature.universe, [25, 50, 75])
temperature['high'] = fuzz.trimf(temperature.universe, [50, 100, 100])

pressure['low'] = fuzz.trimf(pressure.universe, [0, 0, 5])
pressure['medium'] = fuzz.trimf(pressure.universe, [2.5, 5, 7.5])
pressure['high'] = fuzz.trimf(pressure.universe, [5, 10, 10])

reaction_rate['slow'] = fuzz.trimf(reaction_rate.universe, [0, 0, 50])
reaction_rate['optimal'] = fuzz.trimf(reaction_rate.universe, [25, 50, 75])
reaction_rate['fast'] = fuzz.trimf(reaction_rate.universe, [50, 100, 100])

# Define rules
rules = [
    ctrl.Rule(temperature['low'] & pressure['low'], reaction_rate['slow']),
    ctrl.Rule(temperature['medium'] & pressure['medium'], reaction_rate['optimal']),
    ctrl.Rule(temperature['high'] & pressure['high'], reaction_rate['fast']),
    ctrl.Rule(temperature['high'] & pressure['low'], reaction_rate['optimal']),
    ctrl.Rule(temperature['low'] & pressure['high'], reaction_rate['slow']),
]

# Control system
reaction_ctrl = ctrl.ControlSystem(rules)
reaction_sim = ctrl.ControlSystemSimulation(reaction_ctrl)

# Simulation grid
temp_values = np.linspace(0, 100, 50)
press_values = np.linspace(0, 10, 50)
results = np.zeros((50, 50))

# Evaluate controller over range
for i, t in enumerate(temp_values):
    for j, p in enumerate(press_values):
        reaction_sim.input['temperature'] = t
        reaction_sim.input['pressure'] = p
        reaction_sim.compute()
        results[i, j] = reaction_sim.output['reaction_rate']

# Plotting
X, Y = np.meshgrid(press_values, temp_values)
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(X, Y, results, cmap='viridis')
ax.set_xlabel('Pressure')
ax.set_ylabel('Temperature')
ax.set_zlabel('Reaction Rate')
ax.set_title('Fuzzy Logic Reaction Control Surface')
plt.tight_layout()
plt.show()
