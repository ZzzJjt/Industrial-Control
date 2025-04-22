```
import numpy as np
from scipy.linalg import expm

# Constants
feed_rate = 50  # Tons per hour
buffer_tank_capacity = 1000  # Cubic meters (assuming 1 ton = 1 cubic meter for simplicity)
batch_process_time = 2  # Hours (batch process delay)

# Simulation parameters
time_step = 0.1  # Hours (simulation time step)
total_time = 24  # Hours (total simulation time)
num_steps = int(total_time / time_step)

# Initial conditions
material_in_tank = np.zeros(int(batch_process_time / time_step))  # Material in the buffer tank
material_in_tank[-1] = buffer_tank_capacity / 2  # Initial fill level

# Demand profile (fluctuating demand)
demand_profile = np.sin(np.linspace(0, 2 * np.pi, num_steps)) * 20 + 30  # Tons per hour

import cvxpy as cp

# MPC parameters
prediction_horizon = 20  # Prediction horizon in steps
control_horizon = 5  # Control horizon in steps

# States and controls
state = cp.Variable((len(material_in_tank), prediction_horizon))
control = cp.Variable((1, prediction_horizon))

# Constraints
constraints = [
    state[:, 0] == material_in_tank,  # Initial condition
    state >= 0,  # Non-negativity constraint
    state <= buffer_tank_capacity  # Maximum capacity constraint
]

# Objective: Minimize deviation from desired level
desired_level = buffer_tank_capacity * 0.8  # Desired level in the tank
objective = cp.Minimize(cp.sum((state[-1, :] - desired_level)**2))

# Dynamic model
for t in range(prediction_horizon - 1):
    constraints += [state[:, t+1] == state[:, t] + time_step * feed_rate - time_step * demand_profile[t]]

# Solve the optimization problem
prob = cp.Problem(objective, constraints)
results = []

for t in range(num_steps):
    prob.solve()
    results.append(state.value)
    material_in_tank = state.value[:, 0]
    material_in_tank = np.roll(material_in_tank, -1)
    material_in_tank[-1] = material_in_tank[-2] + time_step * feed_rate - time_step * demand_profile[t]
```
