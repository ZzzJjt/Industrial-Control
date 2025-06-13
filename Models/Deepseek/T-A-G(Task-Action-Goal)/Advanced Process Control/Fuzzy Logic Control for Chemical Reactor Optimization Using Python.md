import numpy as np
import matplotlib.pyplot as plt
from skfuzzy import control as ctrl
from scipy.integrate import solve_ivp

# Step 1: Define Key Process Variables
temperature = ctrl.Antecedent(np.arange(-10, 110, 1), 'temperature')
pressure = ctrl.Antecedent(np.arange(0, 201, 1), 'pressure')
reaction_rate = ctrl.Consequent(np.arange(-10, 110, 1), 'reaction_rate')

# Step 2: Create Membership Functions
temperature['low'] = ctrl.trimf(temperature.universe, [-10, -10, 50])
temperature['medium'] = ctrl.trimf(temperature.universe, [30, 70, 110])
temperature['high'] = ctrl.trimf(temperature.universe, [70, 110, 110])

pressure['low'] = ctrl.trimf(pressure.universe, [0, 0, 80])
pressure['medium'] = ctrl.trimf(pressure.universe, [50, 100, 150])
pressure['high'] = ctrl.trimf(pressure.universe, [120, 200, 200])

reaction_rate['decrease'] = ctrl.trimf(reaction_rate.universe, [-10, -10, 40])
reaction_rate['maintain'] = ctrl.trimf(reaction_rate.universe, [20, 50, 80])
reaction_rate['increase'] = ctrl.trimf(reaction_rate.universe, [60, 110, 110])

# Step 3: Develop Fuzzy Logic Rules
rule1 = ctrl.Rule(temperature['low'] & pressure['low'], reaction_rate['increase'])
rule2 = ctrl.Rule(temperature['low'] & pressure['medium'], reaction_rate['increase'])
rule3 = ctrl.Rule(temperature['low'] & pressure['high'], reaction_rate['increase'])

rule4 = ctrl.Rule(temperature['medium'] & pressure['low'], reaction_rate['increase'])
rule5 = ctrl.Rule(temperature['medium'] & pressure['medium'], reaction_rate['maintain'])
rule6 = ctrl.Rule(temperature['medium'] & pressure['high'], reaction_rate['decrease'])

rule7 = ctrl.Rule(temperature['high'] & pressure['low'], reaction_rate['decrease'])
rule8 = ctrl.Rule(temperature['high'] & pressure['medium'], reaction_rate['decrease'])
rule9 = ctrl.Rule(temperature['high'] & pressure['high'], reaction_rate['decrease'])

# Step 4: Implement Fuzzy Inference Engine
reactions_ctrl = ctrl.ControlSystem([rule1, rule2, rule3, rule4, rule5, rule6, rule7, rule8, rule9])
reactions = ctrl.ControlSystemSimulation(reactions_ctrl)

# Step 5: Simulate Chemical Reactor Dynamics
def reactor_dynamics(t, y, u):
    # Simplified nonlinear dynamics
    k_temp = 0.1 + 0.05 * np.sin(t)
    k_press = 0.05 + 0.02 * np.cos(t)
    dydt = -k_temp * y + k_press * u
    return dydt

def simulate_fuzzy_control(setpoint_temp, setpoint_press, initial_condition, t_span, dt):
    t_eval = np.arange(*t_span, dt)
    y = [initial_condition]
    
    for t in t_eval:
        reactions.input['temperature'] = abs(y[-1] - setpoint_temp)  # Error in temperature
        reactions.input['pressure'] = abs(u - setpoint_press)       # Error in pressure
        
        reactions.compute()
        delta_u = reactions.output['reaction_rate']
        
        sol = solve_ivp(fun=lambda t, y: reactor_dynamics(t, y, delta_u), 
                        t_span=(t, t + dt), y0=[y[-1]], t_eval=[t + dt])
        y.append(sol.y.flatten()[-1])
    
    return t_eval, y

# Simulation parameters
setpoint_temp = 50.0
setpoint_press = 100.0
initial_condition = 0.0
u = 50.0  # Initial control input
t_span = (0, 10)
dt = 0.1

# Run simulation
t_fuzzy, y_fuzzy = simulate_fuzzy_control(setpoint_temp, setpoint_press, initial_condition, t_span, dt)

# Step 6: Compare with PID Control
def simulate_pid_control(setpoint_temp, setpoint_press, initial_condition, t_span, dt):
    t_eval = np.arange(*t_span, dt)
    y = [initial_condition]
    integral_temp = 0
    integral_press = 0
    prev_error_temp = 0
    prev_error_press = 0
    
    Kp_temp, Ki_temp, Kd_temp = 1.0, 0.1, 0.05
    Kp_press, Ki_press, Kd_press = 1.0, 0.1, 0.05
    
    for t in t_eval:
        error_temp = setpoint_temp - y[-1]
        integral_temp += error_temp * dt
        derivative_temp = (error_temp - prev_error_temp) / dt
        u_temp = Kp_temp * error_temp + Ki_temp * integral_temp + Kd_temp * derivative_temp
        prev_error_temp = error_temp
        
        error_press = setpoint_press - u
        integral_press += error_press * dt
        derivative_press = (error_press - prev_error_press) / dt
        u_press = Kp_press * error_press + Ki_press * integral_press + Kd_press * derivative_press
        prev_error_press = error_press
        
        u_new = u + u_temp - u_press
        sol = solve_ivp(fun=lambda t, y: reactor_dynamics(t, y, u_new), 
                        t_span=(t, t + dt), y0=[y[-1]], t_eval=[t + dt])
        y.append(sol.y.flatten()[-1])
        u = u_new
    
    return t_eval, y

# Run PID simulation
t_pid, y_pid = simulate_pid_control(setpoint_temp, setpoint_press, initial_condition, t_span, dt)

# Plot results
plt.figure(figsize=(12, 6))

plt.subplot(2, 1, 1)
plt.plot(t_fuzzy, y_fuzzy, label='Fuzzy Controlled')
plt.plot(t_pid, y_pid, label='PID Controlled')
plt.axhline(setpoint_temp, color='r', linestyle='--', label='Setpoint Temp')
plt.title('Temperature Response')
plt.xlabel('Time')
plt.ylabel('Temperature')
plt.legend()

plt.subplot(2, 1, 2)
plt.plot(t_fuzzy, np.full_like(t_fuzzy, u), label='Control Signal (Fuzzy)')
plt.plot(t_pid, np.full_like(t_pid, u), label='Control Signal (PID)', linestyle='--')
plt.title('Control Signals')
plt.xlabel('Time')
plt.ylabel('Control Signal')
plt.legend()

plt.tight_layout()
plt.show()



