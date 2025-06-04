import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize

# Parameters
dt = 0.1                      # Time step
N = 10                        # Prediction horizon
v_max = 1.0                   # Max velocity
goal = np.array([10.0, 10.0]) # Target position

# Obstacle (moving right)
def moving_obstacle(t):
    return np.array([5 + 0.1 * t, 5])

# Robot dynamics: x_{t+1} = x_t + u_t * dt
def dynamics(x, u):
    return x + u * dt

# Cost function
def cost(u_flat, x0, t):
    u = u_flat.reshape((N, 2))
    x = np.copy(x0)
    total_cost = 0.0
    for i in range(N):
        x = dynamics(x, u[i])
        obstacle = moving_obstacle(t + i)
        dist_to_goal = np.linalg.norm(x - goal)
        dist_to_obstacle = np.linalg.norm(x - obstacle)
        total_cost += dist_to_goal
        if dist_to_obstacle < 1.0:
            total_cost += 1000 * (1.0 - dist_to_obstacle)  # Collision penalty
    return total_cost

# MPC controller
def mpc_controller(x0, t):
    u0 = np.zeros((N, 2)).flatten()
    bounds = [(-v_max, v_max)] * 2 * N
    result = minimize(cost, u0, args=(x0, t), bounds=bounds)
    u_opt = result.x.reshape((N, 2))
    return u_opt[0]

# Simulation
x = np.array([0.0, 0.0])
trajectory = [x]
obstacle_traj = []

for t in range(100):
    u = mpc_controller(x, t)
    x = dynamics(x, u)
    trajectory.append(x)
    obstacle_traj.append(moving_obstacle(t))

trajectory = np.array(trajectory)
obstacle_traj = np.array(obstacle_traj)

# Plotting
plt.figure(figsize=(8, 8))
plt.plot(trajectory[:, 0], trajectory[:, 1], label='Robot Path')
plt.plot(goal[0], goal[1], 'go', label='Goal')
plt.plot(obstacle_traj[:, 0], obstacle_traj[:, 1], 'r--', label='Moving Obstacle')
plt.xlim(0, 12)
plt.ylim(0, 12)
plt.xlabel("X")
plt.ylabel("Y")
plt.title("MPC Robot Trajectory with Obstacle Avoidance")
plt.legend()
plt.grid()
plt.show()
