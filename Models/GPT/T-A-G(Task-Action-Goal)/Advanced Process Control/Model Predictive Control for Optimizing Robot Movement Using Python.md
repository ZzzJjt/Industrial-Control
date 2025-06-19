import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize

# Robot motion model (discrete unicycle)
def motion_model(state, control, dt):
    x, y, theta = state
    v, omega = control
    x += v * np.cos(theta) * dt
    y += v * np.sin(theta) * dt
    theta += omega * dt
    return np.array([x, y, theta])

# Cost function for MPC
def cost_function(u_seq, *args):
    state, goal, N, dt, weights = args
    cost = 0
    u_seq = u_seq.reshape((N, 2))
    current = np.copy(state)
    for u in u_seq:
        current = motion_model(current, u, dt)
        pos_error = np.linalg.norm(current[:2] - goal[:2])
        heading_error = abs(current[2] - goal[2])
        cost += weights[0] * pos_error**2 + weights[1] * heading_error**2 + weights[2] * (u[0]**2 + u[1]**2)
    return cost

# Constraints
v_max, omega_max = 1.0, 1.0

# MPC Parameters
N = 10     # Prediction horizon
dt = 0.1   # Time step
weights = [10, 1, 0.1]  # [position error, heading error, control effort]

# Initial and goal states
state = np.array([0.0, 0.0, 0.0])
goal = np.array([5.0, 5.0, np.pi/4])

# Obstacle list (static)
obstacles = [np.array([2.5, 2.5]), np.array([3.5, 4.0])]
radius = 0.5

# Storage for visualization
trajectory = [state.copy()]

# Run MPC loop
for step in range(50):
    u0 = np.zeros(N * 2)  # Initial guess
    bounds = [(-v_max, v_max), (-omega_max, omega_max)] * N

    res = minimize(cost_function, u0, args=(state, goal, N, dt, weights), bounds=bounds)
    if not res.success:
        print("Optimization failed.")
        break

    u_opt = res.x.reshape((N, 2))
    control = u_opt[0]
    state = motion_model(state, control, dt)
    trajectory.append(state.copy())

    # Check for obstacle collision
    for obs in obstacles:
        if np.linalg.norm(state[:2] - obs) < radius:
            print("Collision detected! Aborting.")
            break
    if np.linalg.norm(state[:2] - goal[:2]) < 0.2:
        print("Goal reached.")
        break

# Plot result
trajectory = np.array(trajectory)
plt.figure(figsize=(8, 6))
plt.plot(trajectory[:, 0], trajectory[:, 1], label='Robot Path', marker='o')
plt.plot(goal[0], goal[1], 'ro', label='Goal')
for obs in obstacles:
    circle = plt.Circle(obs, radius, color='gray', alpha=0.5)
    plt.gca().add_patch(circle)
plt.grid(True)
plt.xlabel('X [m]')
plt.ylabel('Y [m]')
plt.legend()
plt.title('MPC Robot Path Planning with Obstacle Avoidance')
plt.axis('equal')
plt.show()
