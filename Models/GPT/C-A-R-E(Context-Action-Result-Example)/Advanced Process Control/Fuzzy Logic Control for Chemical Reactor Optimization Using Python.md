import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt

# Define input and output variables
error = ctrl.Antecedent(np.arange(-10, 11, 1), 'error')  # Current error
delta = ctrl.Antecedent(np.arange(-5, 6, 1), 'delta')    # Change rate of error
u = ctrl.Consequent(np.arange(0, 101, 1), 'control')     # Control output

# Define fuzzy membership functions
error['NB'] = fuzz.trimf(error.universe, [-10, -10, -5])
error['NM'] = fuzz.trimf(error.universe, [-10, -5, 0])
error['Z']  = fuzz.trimf(error.universe, [-2, 0, 2])
error['PM'] = fuzz.trimf(error.universe, [0, 5, 10])
error['PB'] = fuzz.trimf(error.universe, [5, 10, 10])

delta['NB'] = fuzz.trimf(delta.universe, [-5, -5, -2])
delta['NM'] = fuzz.trimf(delta.universe, [-4, -2, 0])
delta['Z']  = fuzz.trimf(delta.universe, [-1, 0, 1])
delta['PM'] = fuzz.trimf(delta.universe, [0, 2, 4])
delta['PB'] = fuzz.trimf(delta.universe, [2, 5, 5])

u['Low']    = fuzz.trimf(u.universe, [0, 0, 40])
u['Medium'] = fuzz.trimf(u.universe, [30, 50, 70])
u['High']   = fuzz.trimf(u.universe, [60, 100, 100])

setpoint = 70
T_fuzzy = 50
T_pid = 50
u_pid = 50
Kp, Ki, Kd = 2.0, 0.1, 0.5
integral = 0
prev_error = 0
T_fuzzy_hist, T_pid_hist = [], []

for t in range(100):
    if t == 50:
        setpoint += 10  # Sudden rise in setpoint to simulate a disturbance

    # Fuzzy control
    err_f = setpoint - T_fuzzy
    d_err_f = err_f - prev_error
    fuzzy_sim.input['error'] = err_f
    fuzzy_sim.input['delta'] = d_err_f
    fuzzy_sim.compute()
    u_fuzzy = fuzzy_sim.output['control']
    T_fuzzy += (u_fuzzy - T_fuzzy) * 0.05  # Simple process model
    T_fuzzy_hist.append(T_fuzzy)

    # PID control
    err = setpoint - T_pid
    integral += err
    derivative = err - prev_error
    u_pid = Kp * err + Ki * integral + Kd * derivative
    T_pid += (u_pid - T_pid) * 0.05
    T_pid_hist.append(T_pid)

    prev_error = err

# Plotting
plt.plot(T_fuzzy_hist, label='Fuzzy Controller')
plt.plot(T_pid_hist, label='PID Controller')
plt.axvline(50, color='r', linestyle='--', label='Disturbance')
plt.title('Fuzzy vs PID Temperature Control')
plt.xlabel('Time Step')
plt.ylabel('Reactor Temperature (°C)')
plt.legend()
plt.grid(True)
plt.show()
