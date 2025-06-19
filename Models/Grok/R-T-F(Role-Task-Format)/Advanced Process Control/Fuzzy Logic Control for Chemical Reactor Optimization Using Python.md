import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy.integrate import odeint
import skfuzzy as fuzz
from skfuzzy import control as ctrl

# Set random seed for reproducibility
np.random.seed(42)

# Step 1: Define reactor dynamics (nonlinear and time-varying)
def reactor_dynamics(state, t, Q, P_valve, C_in, params):
    T, P, C = state  # Temperature (K), Pressure (Pa), Concentration (mol/m^3)
    k0, E, R, deltaH, rho, Cp, V, F = params
    # Reaction rate
    k = k0 * np.exp(-E / (R * T))
    r = k * C
    # Temperature dynamics
    dTdt = (Q - deltaH * r * V - 0.1 * (T - 300)) / (rho * Cp * V)
    # Pressure dynamics (ideal gas approximation)
    dPdt = (R * T * (F * C_in - r * V) - P_valve * P) / V
    # Concentration dynamics
    dCdt = (F * C_in - F * C - r * V) / V
    return [dTdt, dPdt, dCdt]

# Step 2: Define fuzzy logic controller
# Fuzzy variables
error_T = ctrl.Antecedent(np.arange(-50, 51, 1), 'error_T')  # Temp error (K)
error_P = ctrl.Antecedent(np.arange(-5000, 5001, 100), 'error_P')  # Pressure error (Pa)
error_C = ctrl.Antecedent(np.arange(-0.5, 0.51, 0.01), 'error_C')  # Concentration error
Q = ctrl.Consequent(np.arange(0, 10001, 100), 'Q')  # Heat input (W)
P_valve = ctrl.Consequent(np.arange(0, 0.11, 0.001), 'P_valve')  # Valve opening
F = ctrl.Consequent(np.arange(0, 0.21, 0.002), 'F')  # Feed flow (m^3/s)

# Membership functions
error_T['negative'] = fuzz.trimf(error_T.universe, [-50, -25, 0])
error_T['zero'] = fuzz.trimf(error_T.universe, [-10, 0, 10])
error_T['positive'] = fuzz.trimf(error_T.universe, [0, 25, 50])
error_P['negative'] = fuzz.trimf(error_P.universe, [-5000, -2500, 0])
error_P['zero'] = fuzz.trimf(error_P.universe, [-1000, 0, 1000])
error_P['positive'] = fuzz.trimf(error_P.universe, [0, 2500, 5000])
error_C['negative'] = fuzz.trimf(error_C.universe, [-0.5, -0.25, 0])
error_C['zero'] = fuzz.trimf(error_C.universe, [-0.1, 0, 0.1])
error_C['positive'] = fuzz.trimf(error_C.universe, [0, 0.25, 0.5])
Q['low'] = fuzz.trimf(Q.universe, [0, 0, 5000])
Q['medium'] = fuzz.trimf(Q.universe, [2500, 5000, 7500])
Q['high'] = fuzz.trimf(Q.universe, [5000, 10000, 10000])
P_valve['closed'] = fuzz.trimf(P_valve.universe, [0, 0, 0.05])
P_valve['half'] = fuzz.trimf(P_valve.universe, [0.025, 0.05, 0.075])
P_valve['open'] = fuzz.trimf(P_valve.universe, [0.05, 0.1, 0.1])
F['low'] = fuzz.trimf(F.universe, [0, 0, 0.1])
F['medium'] = fuzz.trimf(F.universe, [0.05, 0.1, 0.15])
F['high'] = fuzz.trimf(F.universe, [0.1, 0.2, 0.2])

# Fuzzy rules (example subset for brevity)
rules = [
    ctrl.Rule(error_T['negative'] & error_P['zero'] & error_C['zero'], 
             (Q['high'], P_valve['half'], F['medium'])),
    ctrl.Rule(error_T['positive'] & error_P['zero'] & error_C['zero'], 
             (Q['low'], P_valve['half'], F['medium'])),
    ctrl.Rule(error_P['negative'] & error_T['zero'] & error_C['zero'], 
             (Q['medium'], P_valve['open'], F['medium'])),
    ctrl.Rule(error_P['positive'] & error_T['zero'] & error_C['zero'], 
             (Q['medium'], P_valve['closed'], F['medium'])),
    ctrl.Rule(error_C['negative'] & error_T['zero'] & error_P['zero'], 
             (Q['medium'], P_valve['half'], F['high'])),
    ctrl.Rule(error_C['positive'] & error_T['zero'] & error_P['zero'], 
             (Q['medium'], P_valve['half'], F['low'])),
    # Additional rules for combined errors
    ctrl.Rule(error_T['negative'] & error_P['negative'] & error_C['negative'], 
             (Q['high'], P_valve['open'], F['high'])),
    ctrl.Rule(error_T['positive'] & error_P['positive'] & error_C['positive'], 
             (Q['low'], P_valve['closed'], F['low']))
]

# Fuzzy control system
fuzzy_ctrl = ctrl.ControlSystem(rules)
fuzzy_sim = ctrl.ControlSystemSimulation(fuzzy_ctrl)

# Step 3: Define PID controllers for comparison
def pid_controller(state, setpoints, dt, pid_params, integrals, prev_errors):
    T, P, C = state
    T_sp, P_sp, C_sp = setpoints
    Kp_T, Ki_T, Kd_T, Kp_P, Ki_P, Kd_P, Kp_C, Ki_C, Kd_C = pid_params
    iT, iP, iC = integrals
    eT_prev, eP_prev, eC_prev = prev_errors
    
    # Temperature PID
    eT = T_sp - T
    iT += eT * dt
    dT = (eT - eT_prev) / dt
    Q = Kp_T * eT + Ki_T * iT + Kd_T * dT
    
    # Pressure PID
    eP = P_sp - P
    iP += eP * dt
    dP = (eP - eP_prev) / dt
    P_valve = Kp_P * eP + Ki_P * iP + Kd_P * dP
    
    # Concentration PID
    eC = C_sp - C
    iC += eC * dt
    dC = (eC - eC_prev) / dt
    F = Kp_C * eC + Ki_C * iC + Kd_C * dC
    
    return (np.clip(Q, 0, 10000), np.clip(P_valve, 0, 0.1), np.clip(F, 0, 0.2)), (iT, iP, iC), (eT, eP, eC)

# Step 4: Simulate control performance
def simulate_control(setpoints, controller_type, pid_params=None):
    state = [350, 10000, 1]  # Initial T, P, C
    Q, P_valve, F = 5000, 0.05, 0.1  # Initial controls
    dt = 0.1
    t_sim = np.arange(0, 200, dt)
    history = {'T': [state[0]], 'P': [state[1]], 'C': [state[2]], 
               'Q': [Q], 'P_valve': [P_valve], 'F': [F]}
    params = (1e5, 4e4, 8.314, 5e4, 1000, 1, 1, 0.1)  # Reactor params
    C_in = 2  # Inlet concentration
    integrals = [0, 0, 0]
    prev_errors = [0, 0, 0]
    
    for _ in range(1, len(t_sim)):
        if controller_type == 'fuzzy':
            fuzzy_sim.input['error_T'] = setpoints[0] - state[0]
            fuzzy_sim.input['error_P'] = setpoints[1] - state[1]
            fuzzy_sim.input['error_C'] = setpoints[2] - state[2]
            fuzzy_sim.compute()
            Q = fuzzy_sim.output['Q']
            P_valve = fuzzy_sim.output['P_valve']
            F = fuzzy_sim.output['F']
        else:  # PID
            (Q, P_valve, F), integrals, prev_errors = pid_controller(state, setpoints, dt, pid_params, integrals, prev_errors)
        
        # Update reactor state
        state = odeint(reactor_dynamics, state, [0, dt], args=(Q, P_valve, C_in, params))[-1]
        # Add disturbances
        C_in = 2 + 0.2 * np.sin(0.05 * t_sim[_])
        history['T'].append(state[0])
        history['P'].append(state[1])
        history['C'].append(state[2])
        history['Q'].append(Q)
        history['P_valve'].append(P_valve)
        history['F'].append(F)
    
    return t_sim, history

# Step 5: Run simulations for different scenarios
setpoints = (360, 10000, 1)  # T, P, C setpoints
pid_params = (100, 0.5, 10, 0.001, 0.0001, 0.01, 10, 0.1, 1)  # PID gains
results = {'fuzzy': {}, 'pid': {}}

# Simulate fuzzy and PID control
t_sim, history_fuzzy = simulate_control(setpoints, 'fuzzy')
t_sim, history_pid = simulate_control(setpoints, 'pid', pid_params)
results['fuzzy'] = history_fuzzy
results['pid'] = history_pid

# Step 6: Visualize results
plt.figure(figsize=(12, 10))

# Temperature plot
plt.subplot(3, 1, 1)
plt.plot(t_sim, results['fuzzy']['T'], label='Fuzzy Control')
plt.plot(t_sim, results['pid']['T'], label='PID Control')
plt.axhline(setpoints[0], color='r', linestyle='--', label='Setpoint')
plt.title('Temperature Control')
plt.xlabel('Time (s)')
plt.ylabel('Temperature (K)')
plt.legend()
plt.grid(True)

# Pressure plot
plt.subplot(3, 1, 2)
plt.plot(t_sim, results['fuzzy']['P'], label='Fuzzy Control')
plt.plot(t_sim, results['pid']['P'], label='PID Control')
plt.axhline(setpoints[1], color='r', linestyle='--', label='Setpoint')
plt.title('Pressure Control')
plt.xlabel('Time (s)')
plt.ylabel('Pressure (Pa)')
plt.legend()
plt.grid(True)

# Concentration plot
plt.subplot(3, 1, 3)
plt.plot(t_sim, results['fuzzy']['C'], label='Fuzzy Control')
plt.plot(t_sim, results['pid']['C'], label='PID Control')
plt.axhline(setpoints[2], color='r', linestyle='--', label='Setpoint')
plt.title('Concentration Control')
plt.xlabel('Time (s)')
plt.ylabel('Concentration (mol/m^3)')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.savefig('fuzzy_vs_pid_control.png')

# Step 7: Calculate performance metrics
def calculate_metrics(history, setpoints):
    error_T = np.abs(np.array(history['T']) - setpoints[0])
    error_P = np.abs(np.array(history['P']) - setpoints[1])
    error_C = np.abs(np.array(history['C']) - setpoints[2])
    mse = np.mean([error_T**2, error_P**2, error_C**2])
    settling_time = max(
        next((i * 0.1 for i, e in enumerate(error_T[100:]) if e < 0.01 * setpoints[0]), 200),
        next((i * 0.1 for i, e in enumerate(error_P[100:]) if e < 0.01 * setpoints[1]), 200),
        next((i * 0.1 for i, e in enumerate(error_C[100:]) if e < 0.01 * setpoints[2]), 200)
    )
    return mse, settling_time

mse_fuzzy, st_fuzzy = calculate_metrics(results['fuzzy'], setpoints)
mse_pid, st_pid = calculate_metrics(results['pid'], setpoints)

print("Performance Metrics:")
print(f"Fuzzy - MSE: {mse_fuzzy:.2f}, Settling Time: {st_fuzzy:.1f} s")
print(f"PID - MSE: {mse_pid:.2f}, Settling Time: {st_pid:.1f} s")
