import numpy as np
import matplotlib.pyplot as plt
from casadi import *

# Define the problem dimensions
N = 50  # Number of control intervals
dt = 0.1  # Time step (seconds)
nx = 4  # Number of states (x, y, vx, vy)
nu = 2  # Number of controls (ax, ay)

# State and control bounds
x_min = [-np.inf, -np.inf, -10, -10]
x_max = [np.inf, np.inf, 10, 10]
u_min = [-1, -1]  # Max acceleration
u_max = [1, 1]

# Initial and final states
x0 = [0, 0, 1, 0]  # Start at origin with initial velocity in x-direction
xf = [10, 10, 0, 0]  # Goal position with zero velocity

# Obstacles: list of tuples (x_center, y_center, radius)
obstacles = [(5, 5, 1), (7, 3, 1)]

# Define the symbolic variables
x = MX.sym('x', nx)
u = MX.sym('u', nu)

# Dynamics model: simple kinematic model
f = vertcat(x[2], x[3], u[0], u[1])

# Create an integrator
F = integrator('F', 'rk4', {'x': x, 'p': u, 'ode': f}, {'tf': dt})

# Cost function: quadratic cost on deviation from goal and control effort
Q = diag([1, 1, 0.1, 0.1])  # State weight matrix
R = diag([0.1, 0.1])  # Control weight matrix

# Initialize optimization problem
opti = Opti()

# Decision variables
X = opti.variable(nx, N+1)  # States over prediction horizon
U = opti.variable(nu, N)    # Controls over prediction horizon

# Initial condition
opti.subject_to(X[:, 0] == x0)

# Constraints
for k in range(N):
    X_next = F(x0=X[:, k], p=U[:, k])['xf']
    opti.subject_to(X[:, k+1] == X_next)
    opti.subject_to(opti.bounded(u_min, U[:, k], u_max))
    opti.subject_to(opti.bounded(x_min, X[:, k], x_max))

# Final state constraint
opti.subject_to(opti.bounded(-0.1, X[:2, N] - xf[:2], 0.1))  # Allow small error in reaching goal

# Objective: minimize cost function
obj = 0
for k in range(N):
    obj += mtimes((X[:2, k] - xf[:2]).T, Q[:2, :2], (X[:2, k] - xf[:2])) + \
           mtimes(U[:, k].T, R, U[:, k])
opti.minimize(obj)

# Solve the optimization problem
opti.solver('ipopt')

# Simulation loop
X_sim = np.zeros((nx, N+1))
U_sim = np.zeros((nu, N))
X_sim[:, 0] = x0

plt.figure(figsize=(10, 10))
plt.plot(xf[0], xf[1], 'ro', markersize=10, label='Goal')
for obs in obstacles:
    circle = plt.Circle(obs[:2], obs[2], color='red', alpha=0.3)
    plt.gca().add_patch(circle)

trajectory_x = []
trajectory_y = []

for k in range(N):
    sol = opti.solve()
    X_opt = sol.value(X)
    U_opt = sol.value(U)
    
    X_sim[:, k+1] = X_opt[:, 1]
    U_sim[:, k] = U_opt[:, 0]
    
    trajectory_x.append(X_opt[0, 0])
    trajectory_y.append(X_opt[1, 0])
    
    # Check for collision with obstacles
    collision = False
    for obs in obstacles:
        if np.linalg.norm(np.array([X_opt[0, 0], X_opt[1, 0]]) - np.array(obs[:2])) <= obs[2]:
            collision = True
            break
    
    if collision:
        print("Collision detected!")
        break
    
    # Shift decision variables for next iteration
    opti.set_initial(X[:, :-1], X_opt[:, 1:])
    opti.set_initial(X[:, -1], X_opt[:, -1])
    opti.set_initial(U[:, :-1], U_opt[:, 1:])
    opti.set_initial(U[:, -1], U_opt[:, -1])
    
    # Update plot
    plt.clf()
    plt.plot(xf[0], xf[1], 'ro', markersize=10, label='Goal')
    for obs in obstacles:
        circle = plt.Circle(obs[:2], obs[2], color='red', alpha=0.3)
        plt.gca().add_patch(circle)
    plt.plot(trajectory_x, trajectory_y, '-b', label='Trajectory')
    plt.xlim(0, 10)
    plt.ylim(0, 10)
    plt.xlabel('X Position')
    plt.ylabel('Y Position')
    plt.title('MPC-Controlled Robot Navigation')
    plt.legend()
    plt.pause(0.1)

plt.show()



