import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# 1. Design Fuzzy Logic Rules and Membership Functions
# Input variables
temp = ctrl.Antecedent(np.arange(100, 301, 1), 'temperature')  # °C
pressure = ctrl.Antecedent(np.arange(1, 6.1, 0.1), 'pressure')  # bar
reaction_rate = ctrl.Antecedent(np.arange(0, 2.1, 0.01), 'reaction_rate')  # mol/L/s

# Output variables (control actions)
heater_power = ctrl.Consequent(np.arange(0, 1.1, 0.01), 'heater_power')  # 0–1 (normalized)
pressure_valve = ctrl.Consequent(np.arange(0, 1.1, 0.01), 'pressure_valve')  # 0–1
reactant_flow = ctrl.Consequent(np.arange(0, 1.1, 0.01), 'reactant_flow')  # 0–1

# Membership functions
# Temperature: Low, Medium, High
temp['low'] = fuzz.trimf(temp.universe, [100, 100, 200])
temp['medium'] = fuzz.trimf(temp.universe, [150, 200, 250])
temp['high'] = fuzz.trimf(temp.universe, [200, 300, 300])

# Pressure: Low, Medium, High
pressure['low'] = fuzz.trimf(pressure.universe, [1, 1, 3])
pressure['medium'] = fuzz.trimf(pressure.universe, [2, 3.5, 5])
pressure['high'] = fuzz.trimf(pressure.universe, [4, 6, 6])

# Reaction Rate: Low, Medium, High
reaction_rate['low'] = fuzz.trimf(reaction_rate.universe, [0, 0, 1])
reaction_rate['medium'] = fuzz.trimf(reaction_rate.universe, [0.5, 1, 1.5])
reaction_rate['high'] = fuzz.trimf(reaction_rate.universe, [1, 2, 2])

# Heater Power: Off, Low, Medium, High
heater_power['off'] = fuzz.trimf(heater_power.universe, [0, 0, 0.3])
heater_power['low'] = fuzz.trimf(heater_power.universe, [0.1, 0.3, 0.5])
heater_power['medium'] = fuzz.trimf(heater_power.universe, [0.3, 0.5, 0.7])
heater_power['high'] = fuzz.trimf(heater_power.universe, [0.5, 1, 1])

# Pressure Valve: Closed, Partial, Open
pressure_valve['closed'] = fuzz.trimf(pressure_valve.universe, [0, 0, 0.4])
pressure_valve['partial'] = fuzz.trimf(pressure_valve.universe, [0.2, 0.5, 0.8])
pressure_valve['open'] = fuzz.trimf(pressure_valve.universe, [0.6, 1, 1])

# Reactant Flow: Low, Medium, High
reactant_flow['low'] = fuzz.trimf(reactant_flow.universe, [0, 0, 0.4])
reactant_flow['medium'] = fuzz.trimf(reactant_flow.universe, [0.2, 0.5, 0.8])
reactant_flow['high'] = fuzz.trimf(reactant_flow.universe, [0.6, 1, 1])

# Fuzzy Rules (example subset, total 27 rules for 3^3 combinations simplified)
rules = [
    # Temperature control
    ctrl.Rule(temp['high'] & pressure['medium'] & reaction_rate['medium'], 
              [heater_power['off'], pressure_valve['partial'], reactant_flow['low']]),
    ctrl.Rule(temp['low'] & pressure['medium'] & reaction_rate['medium'], 
              [heater_power['high'], pressure_valve['partial'], reactant_flow['medium']]),
    ctrl.Rule(temp['medium'] & pressure['medium'] & reaction_rate['medium'], 
              [heater_power['medium'], pressure_valve['partial'], reactant_flow['medium']]),
    # Pressure control
    ctrl.Rule(temp['medium'] & pressure['high'] & reaction_rate['medium'], 
              [heater_power['low'], pressure_valve['open'], reactant_flow['low']]),
    ctrl.Rule(temp['medium'] & pressure['low'] & reaction_rate['medium'], 
              [heater_power['medium'], pressure_valve['closed'], reactant_flow['medium']]),
    # Reaction rate control
    ctrl.Rule(temp['medium'] & pressure['medium'] & reaction_rate['high'], 
              [heater_power['medium'], pressure_valve['partial'], reactant_flow['low']]),
    ctrl.Rule(temp['medium'] & pressure['medium'] & reaction_rate['low'], 
              [heater_power['medium'], pressure_valve['partial'], reactant_flow['high']]),
]

# 2. Fuzzy Inference System
fuzzy_system = ctrl.ControlSystem(rules)
controller = ctrl.ControlSystemSimulation(fuzzy_system)

# 3. Simulate Reactor Dynamics
def reactor_model(state, t, u, d):
    """
    Nonlinear reactor model: dT/dt, dP/dt, dR/dt
    T: temperature (°C), P: pressure (bar), R: reaction rate (mol/L/s)
    u: [heater_power, pressure_valve, reactant_flow], d: disturbance
    """
    T, P, R = state
    a = 0.05  # Thermal decay
    b = 50.0  # Heater effect
    c = 0.1   # Pressure coupling
    d_coeff = 0.2  # Disturbance effect
    k = 2.0   # Reaction rate constant
    
    # Nonlinear dynamics
    dTdt = -a * T + b * u[0] + d_coeff * d - 0.01 * T**2 / (T + 100)
    dPdt = c * (R - u[1]) + d_coeff * d
    dRdt = k * u[2] * (1 - R) - 0.1 * R * T / (T + 50)
    
    return [dTdt, dPdt, dRdt]

def simulate_reactor(state0, u_traj, d_traj, dt, t_span):
    states = [state0]
    state = state0
    for i in range(len(t_span)-1):
        t = [t_span[i], t_span[i+1]]
        sol = odeint(reactor_model, state, t, args=(u_traj[i], d_traj[i]))
        state = sol[-1]
        states.append(state)
    return np.array(states)

# Fuzzy Controller
def fuzzy_controller(T, P, R, setpoints):
    controller.inputs({
        'temperature': T,
        'pressure': P,
        'reaction_rate': R
    })
    controller.compute()
    return [
        controller.output['heater_power'],
        controller.output['pressure_valve'],
        controller.output['reactant_flow']
    ]

# PID Controller (for each variable)
class PIDController:
    def __init__(self, Kp=0.5, Ki=0.1, Kd=0.05, dt=0.1):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.dt = dt
        self.integral = 0
        self.prev_error = 0
    
    def compute(self, setpoint, value):
        error = setpoint - value
        self.integral += error * self.dt
        derivative = (error - self.prev_error) / self.dt
        output = self.Kp * error + self.Ki * self.integral + self.Kd * derivative
        self.prev_error = error
        return np.clip(output, 0, 1)

# Simulation Setup
t_span = np.arange(0, 200, 0.1)
setpoints = [200.0, 3.5, 1.0]  # [Temperature (°C), Pressure (bar), Reaction Rate (mol/L/s)]
state0 = [150.0, 2.0, 0.5]  # Initial [T, P, R]

# Disturbance: Step and noise
d_traj = np.ones(len(t_span)) * 0.1
d_traj[500:1000] += 0.2  # Step disturbance
d_traj += np.random.normal(0, 0.02, len(t_span))  # Noise

# Fuzzy Control Simulation
states_fuzzy = [state0]
u_fuzzy = [[0.5, 0.5, 0.5]]
for i in range(1, len(t_span)):
    u = fuzzy_controller(states_fuzzy[-1][0], states_fuzzy[-1][1], states_fuzzy[-1][2], setpoints)
    u_fuzzy.append(u)
    state_next = simulate_reactor(states_fuzzy[-1], [u], [d_traj[i]], 0.1, [t_span[i], t_span[i+1]])[-1]
    states_fuzzy.append(state_next)
states_fuzzy = np.array(states_fuzzy)
u_fuzzy = np.array(u_fuzzy)

# PID Control Simulation
pid_temp = PIDController(Kp=0.5, Ki=0.1, Kd=0.05)
pid_press = PIDController(Kp=0.4, Ki=0.08, Kd=0.04)
pid_rate = PIDController(Kp=0.6, Ki=0.12, Kd=0.06)
states_pid = [state0]
u_pid = [[0.5, 0.5, 0.5]]
for i in range(1, len(t_span)):
    u = [
        pid_temp.compute(setpoints[0], states_pid[-1][0]),
        pid_press.compute(setpoints[1], states_pid[-1][1]),
        pid_rate.compute(setpoints[2], states_pid[-1][2])
    ]
    u_pid.append(u)
    state_next = simulate_reactor(states_pid[-1], [u], [d_traj[i]], 0.1, [t_span[i], t_span[i+1]])[-1]
    states_pid.append(state_next)
states_pid = np.array(states_pid)
u_pid = np.array(u_pid)

# 4. Performance Comparison
def calculate_metrics(states, setpoints, dt):
    errors = setpoints - states
    iae = np.sum(np.abs(errors), axis=0) * dt  # Integral Absolute Error
    overshoot = np.max(np.maximum(0, errors), axis=0)  # Maximum overshoot
    settling_time = np.zeros(3)
    for j in range(3):
        for i in range(len(errors)):
            if abs(errors[i, j]) > 0.02 * setpoints[j] and i * dt > 20:
                settling_time[j] = i * dt
    return iae, overshoot, settling_time

iae_fuzzy, overshoot_fuzzy, settling_fuzzy = calculate_metrics(states_fuzzy, setpoints, 0.1)
iae_pid, overshoot_pid, settling_pid = calculate_metrics(states_pid, setpoints, 0.1)

print("Fuzzy Controller Metrics:")
print(f"IAE: T={iae_fuzzy[0]:.2f}, P={iae_fuzzy[1]:.2f}, R={iae_fuzzy[2]:.2f}")
print(f"Overshoot: T={overshoot_fuzzy[0]:.2f}°C, P={overshoot_fuzzy[1]:.2f}bar, R={overshoot_fuzzy[2]:.2f}mol/L/s")
print(f"Settling Time: T={settling_fuzzy[0]:.2f}s, P={settling_fuzzy[1]:.2f}s, R={settling_fuzzy[2]:.2f}s")
print("\nPID Controller Metrics:")
print(f"IAE: T={iae_pid[0]:.2f}, P={iae_pid[1]:.2f}, R={iae_pid[2]:.2f}")
print(f"Overshoot: T={overshoot_pid[0]:.2f}°C, P={overshoot_pid[1]:.2f}bar, R={overshoot_pid[2]:.2f}mol/L/s")
print(f"Settling Time: T={settling_pid[0]:.2f}s, P={settling_pid[1]:.2f}s, R={settling_pid[2]:.2f}s")

# Plot Results
plt.figure(figsize=(12, 10))
labels = ['Temperature (°C)', 'Pressure (bar)', 'Reaction Rate (mol/L/s)']
for i, title in enumerate(labels):
    plt.subplot(3, 1, i+1)
    plt.plot(t_span, states_fuzzy[:, i], label='Fuzzy')
    plt.plot(t_span, states_pid[:, i], label='PID')
    plt.axhline(setpoints[i], color='r', linestyle='--', label='Setpoint')
    plt.title(title)
    plt.xlabel('Time (s)')
    plt.ylabel(title.split(' ')[0])
    plt.legend()
    plt.grid()

plt.tight_layout()
plt.savefig('reactor_fuzzy_vs_pid.png')
plt.close()
