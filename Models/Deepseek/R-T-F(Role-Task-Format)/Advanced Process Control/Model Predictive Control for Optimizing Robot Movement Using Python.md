import numpy as np
import cvxpy as cp
import matplotlib.pyplot as plt
import time

# Constants
Ts = 0.1  # Sampling time [seconds]
Np = 10   # Prediction horizon
Nu = 2    # Number of control inputs (velocity and angular velocity)
Nx = 3    # Number of states (x, y, theta)

# Motion Model: Kinematic Bicycle Model
def motion_model(x, u):
    """
    x: state vector [x, y, theta]
    u: input vector [v, omega]
    """
    v = u[0]
    omega = u[1]
    theta = x[2]
    
    dx = v * np.cos(theta)
    dy = v * np.sin(theta)
    dtheta = omega
    
    return np.array([dx, dy, dtheta])

# Initialize matrices for QP
Q = np.diag([1.0, 1.0, 0.1])  # State weight matrix
R = np.diag([0.1, 0.1])       # Input weight matrix

# Define obstacles (simple grid-based representation)
obstacles = [(2, 2), (3, 2), (4, 2), (5, 2)]

# Define goal position
goal_position = np.array([8, 8])

# Define initial state
initial_state = np.array([0, 0, 0])

# Define constraints
u_min = np.array([-1.0, -np.pi/4])  # Minimum input values [v_min, omega_min]
u_max = np.array([1.0, np.pi/4])    # Maximum input values [v_max, omega_max]
x_min = np.array([-10, -10, -np.pi])  # Minimum state values [x_min, y_min, theta_min]
x_max = np.array([10, 10, np.pi])     # Maximum state values [x_max, y_max, theta_max]

# Check if a point is inside an obstacle
def is_obstacle(x, y, obstacles, radius=0.5):
    for ox, oy in obstacles:
        if np.sqrt((x - ox)**2 + (y - oy)**2) <= radius:
            return True
    return False

# MPC Controller
class MPCController:
    def __init__(self, nx, nu, np, Q, R, Ts, goal_position, obstacles):
        self.nx = nx
        self.nu = nu
        self.np = np
        self.Q = Q
        self.R = R
        self.Ts = Ts
        self.goal_position = goal_position
        self.obstacles = obstacles

    def compute_control(self, x0):
        # Decision variables
        X = cp.Variable((self.nx, self.np))
        U = cp.Variable((self.nu, self.np-1))

        # Objective function
        objective = 0
        for k in range(self.np):
            objective += cp.quad_form(X[:, k] - self.goal_position, self.Q)
        for k in range(self.np-1):
            objective += cp.quad_form(U[:, k], self.R)

        # Constraints
        constraints = []
        constraints += [X[:, 0] == x0]
        for k in range(self.np-1):
            constraints += [X[:, k+1] == X[:, k] + self.Ts * motion_model(X[:, k], U[:, k])]
            constraints += [U[:, k] >= u_min]
            constraints += [U[:, k] <= u_max]
            constraints += [X[:, k] >= x_min]
            constraints += [X[:, k] <= x_max]
        
        # Obstacle avoidance constraints
        for k in range(self.np):
            for ox, oy in self.obstacles:
                constraints += [cp.norm(X[:2, k] - np.array([ox, oy]), 2) >= 0.5]

        # Problem setup
        prob = cp.Problem(cp.Minimize(objective), constraints)

        # Solve problem
        prob.solve(solver=cp.OSQP)

        # Return first control action
        if prob.status == cp.OPTIMAL:
            return U[:, 0].value
        else:
            print("Optimization failed!")
            return np.zeros(self.nu)

# Simulation Settings
simulation_steps = 100
x_history = [initial_state]
u_history = []

# Create MPC controller
mpc_controller = MPCController(Nx, Nu, Np, Q, R, Ts, goal_position, obstacles)

# Simulation Loop
current_state = initial_state.copy()
for t in range(simulation_steps):
    # Compute control action
    u_optimal = mpc_controller.compute_control(current_state)
    
    # Store history
    x_history.append(current_state)
    u_history.append(u_optimal)
    
    # Update state using motion model
    current_state += Ts * motion_model(current_state, u_optimal)
    
    # Print current state and control action
    print(f"Time: {t*Ts:.2f} s")
    print(f"State: x={current_state[0]:.2f}, y={current_state[1]:.2f}, theta={current_state[2]:.2f}")
    print(f"Control: v={u_optimal[0]:.2f}, omega={u_optimal[1]:.2f}")
    print("----------------------------------------")

# Convert history lists to arrays for plotting
x_history = np.array(x_history)
u_history = np.array(u_history)

# Optional: Plotting
plt.figure(figsize=(12, 6))

# Plot trajectory
plt.subplot(1, 2, 1)
plt.plot(x_history[:, 0], x_history[:, 1], label='Robot Path')
plt.scatter(goal_position[0], goal_position[1], color='green', label='Goal')
for ox, oy in obstacles:
    circle = plt.Circle((ox, oy), 0.5, color='red', alpha=0.5)
    plt.gca().add_patch(circle)
plt.xlabel('X Position')
plt.ylabel('Y Position')
plt.title('Robot Trajectory')
plt.legend()
plt.grid(True)

# Plot controls
plt.subplot(1, 2, 2)
plt.plot(np.arange(simulation_steps) * Ts, u_history[:, 0], label='Velocity (v)')
plt.plot(np.arange(simulation_steps) * Ts, u_history[:, 1], label='Angular Velocity (omega)')
plt.xlabel('Time [s]')
plt.ylabel('Control Inputs')
plt.title('Control Actions')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.show()
