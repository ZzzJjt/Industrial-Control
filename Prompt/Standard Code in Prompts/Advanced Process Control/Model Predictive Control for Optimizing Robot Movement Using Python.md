```
import numpy as np
from scipy.optimize import minimize
import cvxpy as cp

# Robot dynamics model (unicycle model)
def robot_dynamics(x, u, dt):
    """
    Simple unicycle dynamics model.
    x = [x_pos, y_pos, theta]
    u = [v, omega]
    """
    x_next = np.copy(x)
    x_next[0] += u[0] * np.cos(x[2]) * dt
    x_next[1] += u[0] * np.sin(x[2]) * dt
    x_next[2] += u[1] * dt
    return x_next
# Define the objective function
def objective_function(u, *args):
    x, goal, dt, Q, R = args
    x_traj = np.copy(x)
    cost = 0.0
    for i in range(len(u)//2):
        x_traj = robot_dynamics(x_traj, u[i*2:i*2+2], dt)
        cost += (Q @ (x_traj[:2] - goal[:2])) @ (x_traj[:2] - goal[:2])
        cost += R @ u[i*2:i*2+2] @ u[i*2:i*2+2].T
    return cost

# Define the MPC problem using CVXPY
def mpc_controller(x, goal, obstacles, dt, N, Q, R, v_max, omega_max):
    # Variables
    u = cp.Variable((2, N))
    x_pred = cp.Parameter((3, N+1))
    x_pred.value = np.tile(x, (N+1, 1)).T
    
    # Constraints
    constraints = []
    for t in range(N):
        x_pred[:, t+1] = robot_dynamics(x_pred[:, t], u[:, t], dt)
        constraints += [cp.norm_inf(u[:, t]) <= v_max,
                        cp.norm_inf(u[:, t][1]) <= omega_max]
        # Add obstacle avoidance constraints
        for obs in obstacles:
            constraints += [cp.square(x_pred[0, t+1] - obs[0]) + 
                            cp.square(x_pred[1, t+1] - obs[1]) >= obs[2]**2]
    
    # Objective
    obj = cp.Minimize(Q @ cp.sum_squares(x_pred[:2, :] - goal) + R @ cp.sum_squares(u))
    
    # Problem definition
    prob = cp.Problem(obj, constraints)
    
    # Solve the optimization problem
    prob.solve(solver=cp.OSQP)
    
    return u.value[:, 0]
# Simulation parameters
dt = 0.1  # Time step
N = 20  # Prediction horizon
Q = np.eye(2)  # State cost matrix
R = np.eye(2)  # Input cost matrix
v_max = 1.0  # Max velocity
omega_max = np.pi / 4  # Max angular velocity

# Initial conditions
x_init = np.array([0.0, 0.0, 0.0])  # [x, y, theta]
goal = np.array([5.0, 5.0])  # Goal position
obstacles = [(2.5, 2.5, 0.5)]  # Obstacle positions and radii

# Simulation loop
x = x_init
trajectory = [x[:2]]
for t in range(100):
    # Apply MPC controller
    u_opt = mpc_controller(x, goal, obstacles, dt, N, Q, R, v_max, omega_max)
    
    # Update the state
    x = robot_dynamics(x, u_opt, dt)
    
    # Save the state for plotting
    trajectory.append(x[:2])
    
    # Check if goal reached
    if np.linalg.norm(x[:2] - goal) < 0.1:
        break

# Print the trajectory
print("Trajectory:", trajectory)
```
