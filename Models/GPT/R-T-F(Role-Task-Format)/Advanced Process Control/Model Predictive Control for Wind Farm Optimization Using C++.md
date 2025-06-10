import numpy as np
import cvxpy as cp

# Time horizon
N = 24  # 24 hours
dt = 1.0  # timestep in hours

# Wind speed (random with mean 12 m/s and fluctuation)
np.random.seed(0)
wind_speed = 12 + 2 * np.random.randn(N)

# Turbine model parameters
Cp = 0.4  # power coefficient
rho = 1.225  # air density (kg/m³)
A = 100  # swept area (m²)
P_rated = 2e6  # rated power (W)
eta = 0.95  # efficiency

# Storage system
E_max = 5e6  # max energy storage (Wh)
E_init = 2.5e6  # initial energy in storage
charge_eff = 0.95
discharge_eff = 0.95
P_max_storage = 1e6  # max charge/discharge rate (W)

# Demand profile (synthetic sinusoidal)
demand = 1.5e6 + 5e5 * np.sin(np.linspace(0, 2*np.pi, N))  # in W

# Optimization variables
P_turbine = cp.Variable(N)
P_storage = cp.Variable(N)
E_storage = cp.Variable(N)

# Constraints list
constraints = [E_storage[0] == E_init + dt * charge_eff * P_storage[0]]

# Build constraints
for t in range(1, N):
    constraints += [E_storage[t] == E_storage[t-1] + dt * charge_eff * P_storage[t]]
    constraints += [E_storage[t] >= 0, E_storage[t] <= E_max]
    constraints += [cp.abs(P_storage[t]) <= P_max_storage]

for t in range(N):
    wind = wind_speed[t]
    P_available = min(0.5 * rho * A * Cp * wind ** 3 * eta, P_rated)
    constraints += [P_turbine[t] >= 0, P_turbine[t] <= P_available]
    constraints += [P_turbine[t] + P_storage[t] == demand[t]]

# Objective: minimize storage usage and mismatch
objective = cp.Minimize(cp.sum_squares(P_turbine + P_storage - demand) +
                        0.01 * cp.sum_squares(P_storage))

# Solve the problem
problem = cp.Problem(objective, constraints)
problem.solve()

# Print results
print("Hour | Turbine (W) | Storage (W) | Energy (Wh)")
for t in range(N):
    print(f"{t+1:>4} | {P_turbine.value[t]:>11.2f} | {P_storage.value[t]:>10.2f} | {E_storage.value[t]:>10.2f}")
