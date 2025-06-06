import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl

# Define fuzzy variables
temperature = ctrl.Antecedent(np.arange(0, 201, 1), 'temperature')
pressure = ctrl.Antecedent(np.arange(0, 21, 1), 'pressure')
reaction_rate = ctrl.Antecedent(np.arange(0, 11, 1), 'reaction_rate')
control_action = ctrl.Consequent(np.arange(-1, 2, 1), 'control_action')

# Membership functions
temperature['low'] = fuzz.trimf(temperature.universe, [0, 0, 100])
temperature['medium'] = fuzz.trimf(temperature.universe, [80, 100, 130])
temperature['high'] = fuzz.trimf(temperature.universe, [120, 200, 200])

pressure['low'] = fuzz.trimf(pressure.universe, [0, 0, 8])
pressure['normal'] = fuzz.trimf(pressure.universe, [5, 10, 15])
pressure['high'] = fuzz.trimf(pressure.universe, [12, 20, 20])

reaction_rate['slow'] = fuzz.trimf(reaction_rate.universe, [0, 0, 4])
reaction_rate['optimal'] = fuzz.trimf(reaction_rate.universe, [3, 5, 7])
reaction_rate['fast'] = fuzz.trimf(reaction_rate.universe, [6, 10, 10])

control_action['decrease'] = fuzz.trimf(control_action.universe, [-1, -1, 0])
control_action['maintain'] = fuzz.trimf(control_action.universe, [-1, 0, 1])
control_action['increase'] = fuzz.trimf(control_action.universe, [0, 1, 1])

rule1 = ctrl.Rule(temperature['high'] | pressure['high'], control_action['decrease'])
rule2 = ctrl.Rule(temperature['medium'] & pressure['normal'] & reaction_rate['optimal'], control_action['maintain'])
rule3 = ctrl.Rule(temperature['low'] & reaction_rate['slow'], control_action['increase'])

# Build the system
reactor_ctrl = ctrl.ControlSystem([rule1, rule2, rule3])
reactor_sim = ctrl.ControlSystemSimulation(reactor_ctrl)

import matplotlib.pyplot as plt

# Sample disturbance profiles
temp_profile = np.linspace(80, 160, 30)
press_profile = 10 + 3 * np.sin(np.linspace(0, 6, 30))  # varying pressure
rate_profile = 5 + 2 * np.random.randn(30)              # noisy reaction rate

actions = []

for T, P, R in zip(temp_profile, press_profile, rate_profile):
    reactor_sim.input['temperature'] = T
    reactor_sim.input['pressure'] = P
    reactor_sim.input['reaction_rate'] = max(0, min(10, R))  # clamp
    reactor_sim.compute()
    actions.append(reactor_sim.output['control_action'])

# Plotting
plt.plot(actions, label='Fuzzy Control Action')
plt.title('Fuzzy Logic Control Output over Time')
plt.xlabel('Time Step')
plt.ylabel('Control Signal')
plt.grid(True)
plt.legend()
plt.show()

class PID:
    def __init__(self, Kp, Ki, Kd):
        self.Kp, self.Ki, self.Kd = Kp, Ki, Kd
        self.prev_err = 0
        self.integral = 0

    def update(self, setpoint, measured, dt=1.0):
        err = setpoint - measured
        self.integral += err * dt
        deriv = (err - self.prev_err) / dt
        self.prev_err = err
        return self.Kp * err + self.Ki * self.integral + self.Kd * deriv

pid_controller = PID(0.8, 0.1, 0.05)
pid_actions = []

for i in range(30):
    feedback = temp_profile[i] + 0.3 * np.random.randn()
    control = pid_controller.update(120, feedback)
    pid_actions.append(np.clip(control, -1, 1))

# Compare Fuzzy vs PID
plt.plot(actions, label='Fuzzy Controller')
plt.plot(pid_actions, label='PID Controller', linestyle='--')
plt.title('Fuzzy vs PID Control Output')
plt.xlabel('Time Step')
plt.ylabel('Control Signal')
plt.grid(True)
plt.legend()
plt.show()
