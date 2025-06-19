import numpy as np
import matplotlib.pyplot as plt
import casadi as ca

# Parameters
dt = 0.1  # time step [s]
N = 20    # prediction horizon
v_max = 1.0  # max velocity [m/s]
omega_max = np.pi / 4  # max angular velocity [rad/s]
r_min = 0.5  # min turning radius [m]

# Robot initial state: [x, y, theta]
x_init = np.array([0.0, 0.0, 0.0])
goal = np.array([10.0, 10.0])

# Dynamic obstacle (moving)
obstacle_pos = np.array([5.0, 5.0])
obstacle_vel = np.array([0.05, 0.0])
obstacle_radius = 0.5

# MPC variables
opti = ca.Opti()
X = opti.variable(3, N+1)  # state variables
U = opti.variable(2, N)    # control inputs [v, omega]

# Objective
cost = 0
for k in range(N):
    cost += ca.sumsqr(X[0:2, k] - goal[0:2]) + 0.1 * ca.sumsqr(U[:, k])
opti.minimize(cost)

# Dynamics constraint
for k in range(N):
    theta = X[2, k]
    x_next = X[0, k] + U[0, k] * ca.cos(theta) * dt
    y_next = X[1, k] + U[0, k] * ca.sin(theta) * dt
    theta_next = X[2, k] + U[1, k] * dt
    opti.subject_to(X[:, k+1] == ca.vertcat(x_next, y_next, theta_next))

# Input constraints
opti.subject_to(opti.bounded(0, U[0, :], v_max))
opti.subject_to(opti.bounded(-omega_max, U[1, :], omega_max))

# Obstacle avoidance
for k in range(N+1):
    obs = obstacle_pos + k * dt * obstacle_vel
    dist = ca.sqrt((X[0, k] - obs[0])**2 + (X[1, k] - obs[1])**2)
    opti.subject_to(dist >= obstacle_radius + 0.3)

# Initial condition
opti.subject_to(X[:, 0] == x_init)

# Solver
opti.solver("ipopt")
sol = opti.solve()

# Extract and plot result
x_result = sol.value(X)
u_result = sol.value(U)

plt.figure(figsize=(8, 6))
plt.plot(x_result[0, :], x_result[1, :], label='MPC Trajectory')
plt.plot(goal[0], goal[1], 'rx', label='Goal')
for k in range(N+1):
    obs = obstacle_pos + k * dt * obstacle_vel
    circle = plt.Circle(obs, obstacle_radius, color='r', alpha=0.3)
    plt.gca().add_patch(circle)
plt.xlabel('X [m]')
plt.ylabel('Y [m]')
plt.legend()
plt.axis('equal')
plt.title('Mobile Robot Trajectory with MPC and Obstacle Avoidance')
plt.grid()
plt.show()
