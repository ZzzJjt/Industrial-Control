```
import numpy as np
from scipy.integrate import solve_ivp
import cvxpy as cp

# Define the turbine dynamics
def turbine_dynamics(t, state, u):
    # State variables: [temperature, pressure]
    T, P = state
    # Control input: steam flow rate
    F = u
    # Parameters
    alpha = 0.01  # Heat transfer coefficient
    beta = 0.001  # Pressure change coefficient
    dTdt = alpha * F - 0.05 * (T - 300)  # Heat transfer equation
    dPdt = beta * F - 0.005 * P  # Pressure change equation
    return [dTdt, dPdt]

# Initial conditions
T0 = 400  # Initial temperature in Kelvin
P0 = 100  # Initial pressure in bar
state0 = [T0, P0]
# MPC parameters
N = 10  # Prediction horizon
dt = 1  # Time step in seconds
Q = np.diag([1, 1])  # State cost matrix
R = 1  # Control cost matrix

# Define MPC problem
def mpc_controller(T, P, T_ref, P_ref):
    # States and controls
    T_pred = cp.Variable(N)
    P_pred = cp.Variable(N)
    F = cp.Variable(N)  # Control input (steam flow rate)

    # Initial conditions
    T_pred[0] = T
    P_pred[0] = P

    # Constraints
    constraints = [0 <= F, F <= 1000]  # Steam flow rate limits

    # Dynamics constraints
    for k in range(N-1):
        sol = solve_ivp(lambda t, y: turbine_dynamics(t, y, F[k]), [k*dt, (k+1)*dt], [T_pred[k], P_pred[k]])
        T_pred[k+1] = sol.y[0][-1]
        P_pred[k+1] = sol.y[1][-1]
        constraints += [sol.y[0][-1] >= 300, sol.y[0][-1] <= 600]  # Temperature limits
        constraints += [sol.y[1][-1] >= 50, sol.y[1][-1] <= 150]  # Pressure limits

    # Objective function
    obj = cp.sum(Q @ cp.vstack((T_pred - T_ref, P_pred - P_ref))**2) + R * cp.sum(F**2)

    # Problem definition
    prob = cp.Problem(cp.Minimize(obj), constraints)

    # Solve the optimization problem
    prob.solve()

    # Return the first control input
    return F.value[0]
# Simulation parameters
total_time = 100  # Total simulation time in seconds
load_changes = [0, 50, 20, 70]  # Load changes at different times
load_times = [0, 20, 50, 80]  # Times when load changes occur

# Simulation loop
times = np.arange(0, total_time, dt)
T_history = np.zeros_like(times)
P_history = np.zeros_like(times)
F_history = np.zeros_like(times)
T_ref = 450  # Reference temperature
P_ref = 120  # Reference pressure

for i, t in enumerate(times):
    # Determine current load
    if t >= load_times[0]:
        T_ref = load_changes[0]
    if t >= load_times[1]:
        T_ref = load_changes[1]
    if t >= load_times[2]:
        T_ref = load_changes[2]
    if t >= load_times[3]:
        T_ref = load_changes[3]
    
    # Apply MPC controller
    F = mpc_controller(T0, P0, T_ref, P_ref)
    
    # Update the state using the dynamics model
    sol = solve_ivp(lambda t, y: turbine_dynamics(t, y, F), [t, t+dt], [T0, P0])
    T0, P0 = sol.y[:, -1]
    
    # Save the state and control input for plotting
    T_history[i] = T0
    P_history[i] = P0
    F_history[i] = F

# Plot the results
import matplotlib.pyplot as plt

plt.figure()
plt.subplot(2, 1, 1)
plt.plot(times, T_history, label="Temperature")
plt.plot(times, np.full_like(times, T_ref), '--', label="Reference Temperature")
plt.xlabel("Time (s)")
plt.ylabel("Temperature (K)")
plt.legend()

plt.subplot(2, 1, 2)
plt.plot(times, F_history, label="Steam Flow Rate")
plt.xlabel("Time (s)")
plt.ylabel("Flow Rate")
plt.legend()

plt.tight_layout()
plt.show()
```
