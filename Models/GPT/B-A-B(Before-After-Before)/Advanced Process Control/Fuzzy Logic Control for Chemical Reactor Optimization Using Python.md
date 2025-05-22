import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt

# Define fuzzy variables
temperature = ctrl.Antecedent(np.arange(20, 101, 1), 'temperature')
pressure = ctrl.Antecedent(np.arange(1, 11, 0.1), 'pressure')
reaction_rate = ctrl.Consequent(np.arange(0, 101, 1), 'reaction_rate')

# Define fuzzy sets for temperature
temperature['low'] = fuzz.trimf(temperature.universe, [20, 30, 50])
temperature['optimal'] = fuzz.trimf(temperature.universe, [40, 60, 80])
temperature['high'] = fuzz.trimf(temperature.universe, [70, 90, 100])

# Define fuzzy sets for pressure
pressure['low'] = fuzz.trimf(pressure.universe, [1, 2, 5])
pressure['optimal'] = fuzz.trimf(pressure.universe, [4, 6, 8])
pressure['high'] = fuzz.trimf(pressure.universe, [7, 9, 10])

# Define fuzzy sets for reaction rate
reaction_rate['slow'] = fuzz.trimf(reaction_rate.universe, [0, 20, 50])
reaction_rate['medium'] = fuzz.trimf(reaction_rate.universe, [30, 50, 70])
reaction_rate['fast'] = fuzz.trimf(reaction_rate.universe, [60, 80, 100])

# Define fuzzy control rules
rule1 = ctrl.Rule(temperature['low'] | pressure['low'], reaction_rate['slow'])
rule2 = ctrl.Rule(temperature['optimal'] & pressure['optimal'], reaction_rate['medium'])
rule3 = ctrl.Rule(temperature['high'] | pressure['high'], reaction_rate['fast'])

# Build and simulate the fuzzy controller
reactor_ctrl = ctrl.ControlSystem([rule1, rule2, rule3])
reactor_sim = ctrl.ControlSystemSimulation(reactor_ctrl)

# Simulate under varying conditions
temps = np.linspace(20, 100, 50)
pressures = np.linspace(1, 10, 50)
results = []

for t, p in zip(temps, pressures[::-1]):
    reactor_sim.input['temperature'] = t
    reactor_sim.input['pressure'] = p
    reactor_sim.compute()
    results.append((t, p, reactor_sim.output['reaction_rate']))

# Plot result
temps, pressures, rates = zip(*results)
plt.figure(figsize=(10, 6))
plt.plot(temps, rates, label='Reaction Rate')
plt.plot(temps, pressures, '--', label='Pressure (inverted)')
plt.xlabel('Temperature (°C)')
plt.ylabel('Reaction Rate / Pressure')
plt.title('Fuzzy Logic Control of Reactor Dynamics')
plt.legend()
plt.grid(True)
plt.show()
