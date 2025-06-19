import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize

# Robot model: simple unicycle model
def robot_model(x, u, dt):
    x_next = np.zeros_like(x)
    x_next[0] = x[0] + u[0] * np.cos(x[2]) * dt  # x position
    x_next[1] = x[1] + u[0] * np.sin(x[2]) * dt  # y position
    x_next[2] = x[2] + u[1] * dt                 # orientation
    return x_next

# Cost function for MPC
def cost_function(u_flat, *args):
    x0, goal, dt, N, obstacle = args
    u = u_flat.reshape(N, 2)
    x = x0.copy()
    cost = 0.0
    for i in range(N):
        x = robot_model(x, u[i], dt)
        dist_to_goal = np.linalg.norm(x[:2] - goal)
        dist_to_obs = np.linalg.norm(x[:2] - obstacle[:2])
        cost += dist_to_goal**2
        if dist_to_obs < obstacle[2]:  # penalty if inside obstacle radius
            cost += 1000.0 * (obstacle[2] - dist_to_obs)**2
    return cost

# Initial conditions
x0 = np.array([0.0, 0.0, 0.0])   # [x, y, theta]
goal = np.array([10.0, 10.0])    # goal position
obstacle = np.array([5.0, 5.0, 1.0])  # [x, y, radius]
dt = 0.1
N = 10                           # prediction horizon
sim_steps = 100
trajectory = [x0[:2].copy()]

for step in range(sim_steps):
    # Optimization
    u0 = np.zeros((N, 2))  # initial guess
    bounds = [(0, 1), (-np.pi/4, np.pi/4)] * N
    res = minimize(cost_function, u0.flatten(), args=(x0, goal, dt, N, obstacle),
                   bounds=bounds, method='SLSQP')
    if not res.success:
        print("Optimization failed at step", step)
        break

    u_opt = res.x.reshape(N, 2)
    x0 = robot_model(x0, u_opt[0], dt)
    trajectory.append(x0[:2].copy())
    
    # Stop if near goal
    if np.linalg.norm(x0[:2] - goal) < 0.2:
        print("Goal reached at step", step)
        break

# Plot result
trajectory = np.array(trajectory)
plt.figure(figsize=(8, 8))
plt.plot(trajectory[:, 0], trajectory[:, 1], 'b-', label='Trajectory')
plt.plot(goal[0], goal[1], 'go', label='Goal')
circle = plt.Circle(obstacle[:2], obstacle[2], color='r', alpha=0.5, label='Obstacle')
plt.gca().add_artist(circle)
plt.xlabel('X')
plt.ylabel('Y')
plt.title('MPC Robot Path with Obstacle Avoidance')
plt.grid(True)
plt.axis('equal')
plt.legend()
plt.show()
