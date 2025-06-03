import numpy as np
import matplotlib.pyplot as plt
import cvxpy as cp

# Simulation parameters
dt = 0.2  # time step (s)
T_sim = 50  # number of simulation steps
N = 10  # MPC horizon
v_max = 1.0  # max speed (m/s)
a_max = 0.5  # max acceleration (m/s^2)

# Robot dynamics: x = [px, py, vx, vy]
A = np.eye(4)
A[0, 2] = dt
A[1, 3] = dt
B = np.zeros((4, 2))
B[2, 0] = dt
B[3, 1] = dt

# Initial state
x = np.array([0.0, 0.0, 0.0, 0.0])
x_hist = [x.copy()]

# Target point
target = np.array([10.0, 10.0])

# Obstacle (moving)
def obstacle_position(t):
    return np.array([5 + np.sin(0.1 * t), 5 + np.cos(0.1 * t)])

# MPC loop
for t in range(T_sim):
    # Define optimization variables
    u = cp.Variable((N, 2))  # acceleration inputs
    x_var = cp.Variable((N+1, 4))  # state trajectory

    # Cost function
    cost = 0
    constraints = [x_var[0] == x]
    for k in range(N):
        cost += cp.sum_squares(x_var[k, 0:2] - target)  # position cost
        cost += 0.1 * cp.sum_squares(u[k])              # energy use
        # Dynamics constraint
        constraints += [x_var[k+1] == A @ x_var[k] + B @ u[k]]
        # Velocity and acceleration limits
        constraints += [cp.norm(x_var[k+1, 2:4], 2) <= v_max]
        constraints += [cp.norm(u[k], 2) <= a_max]
        # Obstacle avoidance (soft limit)
        obs = obstacle_position(t + k)
        constraints += [cp.norm(x_var[k+1, 0:2] - obs, 2) >= 0.8]

    # Solve MPC problem
    prob = cp.Problem(cp.Minimize(cost), constraints)
    prob.solve()

    # Apply first control input
    u0 = u.value[0]
    x = A @ x + B @ u0
    x_hist.append(x.copy())

# Plotting
x_hist = np.array(x_hist)
plt.figure(figsize=(8, 8))
plt.plot(x_hist[:, 0], x_hist[:, 1], 'b-', label='Robot Path')
plt.plot(target[0], target[1], 'go', label='Target')
for t in range(T_sim):
    obs = obstacle_position(t)
    circle = plt.Circle((obs[0], obs[1]), 0.8, color='r', alpha=0.2)
    plt.gca().add_patch(circle)
plt.xlabel('X'); plt.ylabel('Y')
plt.title('Robot Motion Planning with MPC')
plt.grid(); plt.legend(); plt.axis('equal')
plt.show()
