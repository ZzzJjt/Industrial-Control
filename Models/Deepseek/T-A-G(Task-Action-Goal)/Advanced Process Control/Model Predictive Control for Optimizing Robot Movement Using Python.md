import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize
from shapely.geometry import Point, Polygon

# Step 1: Develop the Motion Model
def motion_model(x, u, dt):
    """
    Kinematic model of the robot.
    
    Parameters:
        x (numpy.ndarray): Current state [x, y, theta].
        u (numpy.ndarray): Control input [v, omega].
        dt (float): Time step.
    
    Returns:
        numpy.ndarray: Next state [x_next, y_next, theta_next].
    """
    v, omega = u
    theta = x[2]
    x_next = x[0] + v * np.cos(theta) * dt
    y_next = x[1] + v * np.sin(theta) * dt
    theta_next = theta + omega * dt
    return np.array([x_next, y_next, theta_next])

# Step 2: Implement the MPC Algorithm
class MPCController:
    def __init__(self, N, dt, Q, R, x_min, x_max, u_min, u_max, obstacle_list):
        self.N = N  # Prediction horizon
        self.dt = dt  # Time step
        self.Q = Q  # State cost matrix
        self.R = R  # Input cost matrix
        self.x_min = x_min  # State lower bounds
        self.x_max = x_max  # State upper bounds
        self.u_min = u_min  # Input lower bounds
        self.u_max = u_max  # Input upper bounds
        self.obstacle_list = obstacle_list  # List of obstacles

    def predict(self, x0, u_seq):
        """
        Predict future states given initial state and control sequence.
        
        Parameters:
            x0 (numpy.ndarray): Initial state [x, y, theta].
            u_seq (numpy.ndarray): Sequence of controls [v, omega].
        
        Returns:
            numpy.ndarray: Predicted states over the prediction horizon.
        """
        x_pred = np.zeros((self.N, 3))
        x_pred[0] = x0
        for k in range(1, self.N):
            x_pred[k] = motion_model(x_pred[k-1], u_seq[k-1], self.dt)
        return x_pred

    def collision_check(self, x, obstacles):
        """
        Check for collisions between the robot and obstacles.
        
        Parameters:
            x (numpy.ndarray): State [x, y, theta].
            obstacles (list): List of Shapely Polygons representing obstacles.
        
        Returns:
            bool: True if there is no collision, False otherwise.
        """
        robot_point = Point(x[0], x[1])
        for obstacle in obstacles:
            if obstacle.contains(robot_point):
                return False
        return True

    def objective_function(self, u_flat, x0, goal):
        """
        Objective function for optimization.
        
        Parameters:
            u_flat (numpy.ndarray): Flattened control sequence.
            x0 (numpy.ndarray): Initial state [x, y, theta].
            goal (numpy.ndarray): Goal position [x_goal, y_goal].
        
        Returns:
            float: Cost associated with the control sequence.
        """
        u_seq = u_flat.reshape((self.N, 2))
        x_pred = self.predict(x0, u_seq)
        cost = 0.0
        for k in range(self.N):
            cost += self.Q @ (x_pred[k][:2] - goal).T @ (x_pred[k][:2] - goal)
            cost += self.R @ u_seq[k].T @ u_seq[k]
            
            # Penalty for collision
            if not self.collision_check(x_pred[k], self.obstacle_list):
                cost += 1e6  # High penalty for collision
        
        return cost

    def optimize(self, x0, goal):
        """
        Optimize the control sequence using nonlinear optimization.
        
        Parameters:
            x0 (numpy.ndarray): Initial state [x, y, theta].
            goal (numpy.ndarray): Goal position [x_goal, y_goal].
        
        Returns:
            numpy.ndarray: Optimized control sequence.
        """
        u_flat0 = np.zeros(2 * self.N)
        bounds = [(u_min, u_max) for _ in range(2 * self.N)]
        result = minimize(self.objective_function, u_flat0, args=(x0, goal), method='SLSQP', bounds=bounds)
        u_opt = result.x.reshape((self.N, 2))
        return u_opt

# Step 3: Simulate Various Environments
def simulate_environment(x0, goal, mpc_controller, simulation_time, dt, obstacles):
    """
    Simulate the robot's movement in a dynamic environment.
    
    Parameters:
        x0 (numpy.ndarray): Initial state [x, y, theta].
        goal (numpy.ndarray): Goal position [x_goal, y_goal].
        mpc_controller (MPCController): MPC controller instance.
        simulation_time (float): Total simulation time.
        dt (float): Time step.
        obstacles (list): List of Shapely Polygons representing obstacles.
    
    Returns:
        list: Trajectory of the robot.
    """
    num_steps = int(simulation_time / dt)
    trajectory = [x0.copy()]
    current_state = x0.copy()

    for t in range(num_steps):
        u_opt = mpc_controller.optimize(current_state, goal)
        next_state = motion_model(current_state, u_opt[0], dt)
        
        # Check for collision
        if not mpc_controller.collision_check(next_state, obstacles):
            print("Collision detected! Stopping simulation.")
            break
        
        trajectory.append(next_state)
        current_state = next_state
    
    return trajectory

# Define parameters
N = 10  # Prediction horizon
dt = 0.5  # Time step (seconds)
Q = np.diag([1.0, 1.0])  # State cost matrix
R = np.diag([0.1, 0.1])  # Input cost matrix
x_min = np.array([-np.inf, -np.inf, -np.pi])
x_max = np.array([np.inf, np.inf, np.pi])
u_min = np.array([0.0, -np.pi/4])  # Minimum velocity and angular velocity
u_max = np.array([1.0, np.pi/4])   # Maximum velocity and angular velocity

# Define obstacles
obstacles = [
    Polygon([(1, 1), (2, 1), (2, 2), (1, 2)]),
    Polygon([(5, 3), (6, 3), (6, 4), (5, 4)])
]

# Initialize MPC controller
mpc_controller = MPCController(N, dt, Q, R, x_min, x_max, u_min, u_max, obstacles)

# Simulation parameters
initial_state = np.array([0.0, 0.0, 0.0])  # Initial state [x, y, theta]
goal_position = np.array([8.0, 8.0])         # Goal position [x_goal, y_goal]
simulation_time = 20                         # Total simulation time (seconds)

# Run simulation
trajectory = simulate_environment(initial_state, goal_position, mpc_controller, simulation_time, dt, obstacles)

# Plot results
plt.figure(figsize=(10, 10))

# Plot trajectory
trajectory_array = np.array(trajectory)
plt.plot(trajectory_array[:, 0], trajectory_array[:, 1], '-o', label='Robot Trajectory')

# Plot start and goal positions
plt.scatter(initial_state[0], initial_state[1], c='green', s=100, label='Start')
plt.scatter(goal_position[0], goal_position[1], c='red', s=100, label='Goal')

# Plot obstacles
for obstacle in obstacles:
    x, y = obstacle.exterior.xy
    plt.fill(x, y, alpha=0.5, fc='gray', ec='none', label='Obstacle' if not obstacle in obstacles[:-1] else None)

plt.title('Robot Navigation with MPC')
plt.xlabel('X Position')
plt.ylabel('Y Position')
plt.legend()
plt.grid(True)
plt.axis('equal')
plt.show()



