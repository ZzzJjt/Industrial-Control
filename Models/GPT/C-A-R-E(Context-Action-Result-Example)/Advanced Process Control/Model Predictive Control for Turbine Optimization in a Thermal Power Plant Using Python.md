import numpy as np
from scipy.optimize import minimize
import matplotlib.pyplot as plt

# Parameters
dt = 1.0            # Time step (s)
N = 10              # Prediction horizon
T_max = 800         # Max allowable temperature (°C)
u_max = 100.0       # Max fuel input
u_min = 0.0         # Min fuel input
T0 = 600            # Initial temperature
T_set_base = 650    # Base setpoint

# Turbine dynamics: T[k+1] = T[k] + (0.1 * u[k] - 0.05 * (T[k] - ambient)) * dt
def turbine_dynamics(T, u, ambient=25.0):
    return T + (0.1 * u - 0.05 * (T - ambient)) * dt

# MPC cost function
def cost(u_flat, T_init, T_set):
    u = u_flat.reshape(N)
    T = T_init
    total_cost = 0.0
    for i in range(N):
        T = turbine_dynamics(T, u[i])
        error = T - T_set
        total_cost += error**2 + 0.01 * (u[i]**2)  # penalize large control input
    return total_cost

# MPC controller
def mpc_controller(T_current, T_set):
    u0 = np.full(N, 50.0)  # initial guess
    bounds = [(u_min, u_max)] * N
    result = minimize(cost, u0, args=(T_current, T_set), bounds=bounds)
    return result.x[0] if result.success else 50.0

# Simulation loop
time_steps = 100
T = T0
T_history = []
T_set_history = []
u_history = []

for k in range(time_steps):
    # Simulate demand surge at t=50
    T_set = T_set_base + (20 if k > 50 else 0)
    u = mpc_controller(T, T_set)
    T = turbine_dynamics(T, u)
    
    # Store results
    T_history.append(T)
    T_set_history.append(T_set)
    u_history.append(u)

# Plot results
plt.figure(figsize=(10, 5))
plt.subplot(2, 1, 1)
plt.plot(T_history, label='Turbine Temperature')
plt.plot(T_set_history, '--', label='Setpoint')
plt.ylabel("Temperature (°C)")
plt.legend()
plt.grid()

plt.subplot(2, 1, 2)
plt.plot(u_history, label='Fuel Input Rate (u)')
plt.xlabel("Time Step")
plt.ylabel("Control Input")
plt.legend()
plt.grid()

plt.tight_layout()
plt.show()
