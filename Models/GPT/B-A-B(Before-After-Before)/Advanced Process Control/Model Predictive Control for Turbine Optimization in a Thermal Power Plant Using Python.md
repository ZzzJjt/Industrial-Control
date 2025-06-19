import numpy as np
import matplotlib.pyplot as plt
import cvxpy as cp

# Simulation parameters
dt = 1.0           # time step [s]
T_sim = 200        # total simulation time [s]
N = 20             # MPC prediction horizon

# System dynamics (simplified turbine thermal + power model)
# States: [temperature, power output]
A = np.array([[0.98, 0.01],
              [0.05, 0.95]])
B = np.array([[0.05],
              [0.1]])

# Control input: fuel valve position (0-1)
u_min, u_max = 0.0, 1.0

# State constraints
temp_max = 650.0  # °C
power_max = 500.0  # MW

# Setpoint: desired load demand (MW)
load_demand = 300 + 100 * np.sin(np.linspace(0, 2*np.pi, T_sim))

# Disturbance profile (external cooling loss, sudden demand surge)
def disturbance(t):
    return np.array([10.0 * np.sin(0.05*t), 0.0]) if 50 <= t <= 150 else np.zeros(2)

# Initial state
x = np.array([500.0, 300.0])
x_hist = [x.copy()]
u_hist = []
demand_track = []

# MPC loop
for t in range(T_sim):
    x0 = x.copy()
    ref = np.array([600.0, load_demand[t]])  # target temperature & power

    # MPC variables
    u = cp.Variable((N, 1))
    x_var = cp.Variable((N+1, 2))
    cost = 0
    constraints = [x_var[0] == x0]

    for k in range(N):
        cost += 10 * cp.sum_squares(x_var[k+1, 1] - ref[1])   # power tracking
        cost += 1 * cp.sum_squares(x_var[k+1, 0] - ref[0])     # temp stability
        cost += 0.1 * cp.sum_squares(u[k])                    # energy use

        # System dynamics
        constraints += [x_var[k+1] == A @ x_var[k] + B @ u[k]]

        # Input constraints
        constraints += [u[k] >= u_min, u[k] <= u_max]

        # State constraints
        constraints += [x_var[k+1, 0] <= temp_max]
        constraints += [x_var[k+1, 1] <= power_max]

    # Solve
    prob = cp.Problem(cp.Minimize(cost), constraints)
    prob.solve(solver=cp.OSQP)

    # Apply control
    u_apply = u.value[0, 0]
    x = A @ x + B.flatten() * u_apply + disturbance(t)
    x_hist.append(x.copy())
    u_hist.append(u_apply)
    demand_track.append(load_demand[t])

# Convert to arrays
x_hist = np.array(x_hist)
u_hist = np.array(u_hist)
demand_track = np.array(demand_track)

# Plot results
time = np.arange(T_sim+1)

plt.figure(figsize=(12, 6))
plt.subplot(3,1,1)
plt.plot(time, x_hist[:,0], label='Temperature [°C]')
plt.axhline(temp_max, color='r', linestyle='--')
plt.ylabel('Temp [°C]')
plt.grid(); plt.legend()

plt.subplot(3,1,2)
plt.plot(time, x_hist[:,1], label='Power Output [MW]')
plt.plot(time[:-1], demand_track, 'r--', label='Load Demand')
plt.axhline(power_max, color='k', linestyle='--')
plt.ylabel('Power [MW]')
plt.grid(); plt.legend()

plt.subplot(3,1,3)
plt.step(time[:-1], u_hist, label='Fuel Valve (u)')
plt.ylabel('Fuel Valve'); plt.xlabel('Time [s]')
plt.grid(); plt.legend()

plt.tight_layout()
plt.show()
