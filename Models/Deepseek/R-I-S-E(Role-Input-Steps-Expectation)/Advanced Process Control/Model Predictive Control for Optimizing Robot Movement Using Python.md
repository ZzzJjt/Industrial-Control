import numpy as np
from casadi import *

# System parameters
dt = 0.1        # Time step
N = 20          # Prediction horizon
v_max = 1.5     # m/s
v_min = 0.0
omega_max = np.pi / 2  # rad/s
a_max = 1.0
j_max = 3.0

# Unicycle dynamics function
def unicycle_dynamics(x, u):
    px, py, theta, v, omega = x[0], x[1], x[2], x[3], x[4]
    dv, domega = u[0], u[1]

    new_v = v + dv * dt
    new_omega = omega + domega * dt

    new_px = px + v * cos(theta) * dt
    new_py = py + v * sin(theta) * dt
    new_theta = theta + omega * dt

    return [new_px, new_py, new_theta, new_v, new_omega]

opti = Opti()

# Decision variables
X = opti.variable(N+1, 5)   # States over prediction horizon
U = opti.variable(N, 2)     # Inputs over prediction horizon

# Parameters (for setting references and obstacle positions)
P_goal = opti.parameter(2)         # Goal position
P_obstacles = opti.parameter(2, 5) # Up to 5 obstacles (x,y)

# Initial state
X0 = opti.parameter(5)
opti.subject_to(X[0, :] == X0)

# Dynamics constraints
for k in range(N):
    x_next = unicycle_dynamics(X[k, :], U[k, :])
    opti.subject_to(X[k+1, :] == x_next)

# State constraints
for k in range(N+1):
    opti.subject_to(opti.bounded(v_min, X[k, 3], v_max))
    opti.subject_to(opti.bounded(-omega_max, X[k, 4], omega_max))

# Input constraints
for k in range(N):
    opti.subject_to(opti.bounded(-a_max, U[k, 0], a_max))      # Acceleration limits
    opti.subject_to(opti.bounded(-j_max, U[k, 1], j_max))      # Angular jerk limits

# Obstacle avoidance constraints
for k in range(N+1):
    for i in range(P_obstacles.shape[1]):
        dx = X[k, 0] - P_obstacles[0, i]
        dy = X[k, 1] - P_obstacles[1, i]
        dist = sqrt(dx**2 + dy**2)
        safety_radius = 0.5
        opti.subject_to(dist >= safety_radius)

# Objective function
cost = 0
for k in range(N):
    dxg = X[k, 0] - P_goal[0]
    dyg = X[k, 1] - P_goal[1]
    cost += dxg**2 + dyg**2 + 0.1 * U[k, 0]**2 + 0.05 * U[k, 1]**2

opti.minimize(cost)

# Set solver options
opti.solver('ipopt')

# Simulation setup
T_sim = 30
sim_steps = int(T_sim / dt)

# Initial state
state = np.array([0, 0, 0, 0, 0])

# Store history
history = []

# Set initial obstacles
obstacles = np.array([
    [4, 7, 10, 13, 15],
    [4, 6, 8, 6, 4]
])

# Solve MPC and apply first control input
for t in range(sim_steps):
    if t % 100 == 0:
        # Update goal point every 10 seconds
        goal_idx = int(t / 100)
        goal_points = [[10, 10], [8, 6], [5, 10]]
        current_goal = goal_points[goal_idx % len(goal_points)]

        # Move obstacle slightly (simulate dynamic environment)
        obstacles[0, :] += np.random.uniform(-0.05, 0.05)
        obstacles[1, :] += np.random.uniform(-0.05, 0.05)

    # Set parameter values
    opti.set_value(X0, state)
    opti.set_value(P_goal, current_goal)
    opti.set_value(P_obstacles, obstacles)

    try:
        sol = opti.solve()
        u_opt = sol.value(U)[0]
    except:
        u_opt = [0, 0]  # Fallback on failure

    # Apply control and integrate dynamics forward
    state = unicycle_dynamics(state, u_opt)

    # Log data
    history.append(state.copy())

# Convert to numpy array for analysis
history = np.array(history)

# Final position
final_position = history[-1, :2]
goal_reached = np.linalg.norm(final_position - current_goal) < 0.5

# Path length
path_length = np.sum(np.linalg.norm(np.diff(history[:, :2], axis=0), axis=1))

# Efficiency metric
straight_line_distance = np.linalg.norm(current_goal - [0, 0])
efficiency = straight_line_distance / path_length

# Print results
print(f"Goal reached: {goal_reached}")
print(f"Path length: {path_length:.2f} m")
print(f"Efficiency (ratio): {efficiency:.2f}")
print(f"Final heading: {np.degrees(history[-1, 2]):.1f}°")
