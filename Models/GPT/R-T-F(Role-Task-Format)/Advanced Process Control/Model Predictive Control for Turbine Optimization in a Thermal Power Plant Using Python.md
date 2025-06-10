import numpy as np
import cvxpy as cp

# Simulation parameters
time_horizon = 50        # number of time steps
delta_t = 1.0            # time step size (minutes)
prediction_horizon = 10  # MPC prediction horizon

# Turbine parameters (simplified thermal model)
tau = 10.0               # thermal time constant
K = 1.0                  # turbine gain
T_ambient = 25.0         # ambient temperature
efficiency_target = 100  # target output power (MW)

# Constraints
T_min = 30.0             # min turbine inlet temp (°C)
T_max = 100.0            # max turbine inlet temp (°C)
u_min = 0.0              # min heat input
u_max = 10.0             # max heat input

# Initial conditions
T = 30.0                 # turbine inlet temperature
state_log = []
power_log = []
control_log = []

# External load disturbance (simulated demand fluctuations)
np.random.seed(42)
external_load = efficiency_target + 10 * np.sin(np.linspace(0, 2*np.pi, time_horizon)) + 5*np.random.randn(time_horizon)

# System dynamics: T_next = T + delta_t * ( -T / tau + K * u / tau )
def simulate_turbine(T, u):
    return T + delta_t * (-T / tau + K * u / tau)

# MPC optimization loop
for t in range(time_horizon - prediction_horizon):
    # Variables
    u = cp.Variable(prediction_horizon)
    T_pred = cp.Variable(prediction_horizon + 1)
    
    # Constraints and objective
    constraints = [T_pred[0] == T]
    obj = 0
    for k in range(prediction_horizon):
        T_next = T_pred[k] + delta_t * (-T_pred[k] / tau + K * u[k] / tau)
        constraints += [
            T_pred[k+1] == T_next,
            T_pred[k+1] >= T_min,
            T_pred[k+1] <= T_max,
            u[k] >= u_min,
            u[k] <= u_max
        ]
        # Power output ~ temperature; deviation from desired power
        power_pred = T_pred[k+1]
        load_target = external_load[t + k]
        obj += cp.square(power_pred - load_target)
    
    # Solve
    prob = cp.Problem(cp.Minimize(obj), constraints)
    prob.solve()

    # Apply first control move
    u_apply = u.value[0]
    T = simulate_turbine(T, u_apply)

    # Log data
    control_log.append(u_apply)
    state_log.append(T)
    power_log.append(K * T)  # Output power is proportional to T

# Display output logs
for i in range(len(state_log)):
    print(f"Step {i:02d} | Control Input: {control_log[i]:.2f} | Temp: {state_log[i]:.2f}°C | Power Output: {power_log[i]:.2f} MW")
