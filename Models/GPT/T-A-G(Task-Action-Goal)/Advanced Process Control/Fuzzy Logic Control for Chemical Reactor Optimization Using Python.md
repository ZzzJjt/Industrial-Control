import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt

# Define fuzzy variables
Temperature = ctrl.Antecedent(np.arange(0, 101, 1), 'Temperature')
Pressure = ctrl.Antecedent(np.arange(0, 11, 1), 'Pressure')
ReactionRate = ctrl.Consequent(np.arange(0, 101, 1), 'ReactionRate')

# Membership functions for Temperature
Temperature['low'] = fuzz.trimf(Temperature.universe, [0, 0, 50])
Temperature['medium'] = fuzz.trimf(Temperature.universe, [30, 50, 70])
Temperature['high'] = fuzz.trimf(Temperature.universe, [50, 100, 100])

# Membership functions for Pressure
Pressure['low'] = fuzz.trimf(Pressure.universe, [0, 0, 5])
Pressure['medium'] = fuzz.trimf(Pressure.universe, [3, 5, 7])
Pressure['high'] = fuzz.trimf(Pressure.universe, [5, 10, 10])

# Membership functions for Reaction Rate
ReactionRate['slow'] = fuzz.trimf(ReactionRate.universe, [0, 0, 50])
ReactionRate['moderate'] = fuzz.trimf(ReactionRate.universe, [25, 50, 75])
ReactionRate['fast'] = fuzz.trimf(ReactionRate.universe, [50, 100, 100])

# Define fuzzy rules
rule1 = ctrl.Rule(Temperature['high'] & Pressure['high'], ReactionRate['fast'])
rule2 = ctrl.Rule(Temperature['medium'] & Pressure['medium'], ReactionRate['moderate'])
rule3 = ctrl.Rule(Temperature['low'] | Pressure['low'], ReactionRate['slow'])
rule4 = ctrl.Rule(Temperature['high'] & Pressure['low'], ReactionRate['moderate'])
rule5 = ctrl.Rule(Temperature['low'] & Pressure['high'], ReactionRate['moderate'])

# Build control system
reaction_ctrl = ctrl.ControlSystem([rule1, rule2, rule3, rule4, rule5])
reaction_sim = ctrl.ControlSystemSimulation(reaction_ctrl)

# Simulate reactor operation
sample_temps = np.linspace(20, 80, 30)
sample_pressures = np.linspace(2, 8, 30)

reaction_rates = []
for t, p in zip(sample_temps, sample_pressures):
    reaction_sim.input['Temperature'] = t
    reaction_sim.input['Pressure'] = p
    reaction_sim.compute()
    reaction_rates.append(reaction_sim.output['ReactionRate'])

# Plot results
plt.figure(figsize=(10, 6))
plt.plot(reaction_rates, label='Fuzzy Logic Reaction Rate', marker='o')
plt.xlabel('Sample Index')
plt.ylabel('Reaction Rate')
plt.title('Fuzzy Logic Control of Chemical Reactor')
plt.legend()
plt.grid(True)
plt.show()
