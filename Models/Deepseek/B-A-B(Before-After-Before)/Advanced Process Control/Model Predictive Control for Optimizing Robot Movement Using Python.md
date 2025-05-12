import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Circle
from scipy.optimize import minimize

class RobotMPC:
    def __init__(self):
        # Robot physical parameters
        self.length = 0.5  # Robot length [m]
        self.width = 0.3   # Robot width [m]
        self.max_speed = 1.0  # m/s
        self.max_steering = np.pi/4  # rad
        self.max_accel = 0.5  # m/s²
        self.max_steering_rate = np.pi/8  # rad/s
        
        # MPC parameters
        self.dt = 0.1  # Time step [s]
        self.horizon = 10  # Prediction horizon steps
        self.obstacle_margin = 0.5  # Safety margin around obstacles [m]
        
        # Cost function weights
        self.w_pos = 1.0    # Target position weight
        self.w_vel = 0.1     # Velocity smoothness weight
        self.w_steer = 0.1   # Steering effort weight
        self.w_obs = 5.0     # Obstacle avoidance weight
        
        # Current state [x, y, theta, v, steering_angle]
        self.state = np.array([0.0, 0.0, 0.0, 0.0, 0.0])
        
        # Dynamic obstacles (x, y, radius, vx, vy)
        self.obstacles = np.array([
            [3.0, 2.0, 0.4, 0.2, -0.1],
            [1.5, 3.0, 0.3, -0.1, -0.2],
            [4.0, 1.0, 0.5, 0.0, 0.1]
        ])
    
    def bicycle_model(self, state, control):
        """Kinematic bicycle model for robot dynamics"""
        x, y, theta, v, steer = state
        accel, steer_rate = control
        
        # Apply control constraints
        accel = np.clip(accel, -self.max_accel, self.max_accel)
        steer_rate = np.clip(steer_rate, -self.max_steering_rate, self.max_steering_rate)
        
        # Update steering angle
        new_steer = steer + steer_rate * self.dt
        new_steer = np.clip(new_steer, -self.max_steering, self.max_steering)
        
        # Update velocity
        new_v = v + accel * self.dt
        new_v = np.clip(new_v, 0, self.max_speed)
        
        # Update position and orientation
        new_theta = theta + (new_v * np.tan(new_steer) / self.length) * self.dt
        new_x = x + new_v * np.cos(new_theta) * self.dt
        new_y = y + new_v * np.sin(new_theta) * self.dt
        
        return np.array([new_x, new_y, new_theta, new_v, new_steer])
    
    def predict_trajectory(self, init_state, controls):
        """Predict future trajectory given initial state and control sequence"""
        trajectory = []
        state = init_state.copy()
        trajectory.append(state)
        
        # Apply each control in sequence
        for i in range(self.horizon):
            control = controls[i*2:(i+1)*2]  # Get (accel, steer_rate) pair
            state = self.bicycle_model(state, control)
            trajectory.append(state)
            
        return np.array(trajectory)
    
    def cost_function(self, controls, target):
        """Calculate MPC cost for given control sequence"""
        cost = 0.0
        trajectory = self.predict_trajectory(self.state, controls)
        
        # Terminal state cost (distance to target)
        final_pos = trajectory[-1, :2]
        cost += self.w_pos * np.linalg.norm(final_pos - target)
        
        # Control smoothness costs
        for i in range(self.horizon):
            # Velocity cost (encourage maintaining reasonable speed)
            cost += self.w_vel * (trajectory[i, 3] - self.max_speed/2)**2
            
            # Steering effort cost
            cost += self.w_steer * (controls[i*2+1]**2)  # steer_rate
            
            # Obstacle avoidance cost
            for obs in self.obstacles:
                obs_pos = obs[:2] + obs[3:5] * i * self.dt  # Moving obstacle
                dist = np.linalg.norm(trajectory[i, :2] - obs_pos)
                safe_dist = obs[2] + self.obstacle_margin
                if dist < safe_dist:
                    cost += self.w_obs * (1.0 - dist/safe_dist)**2
        
        return cost
    
    def update_obstacles(self):
        """Simulate dynamic obstacle movement"""
        for i in range(len(self.obstacles)):
            self.obstacles[i, 0] += self.obstacles[i, 3] * self.dt
            self.obstacles[i, 1] += self.obstacles[i, 4] * self.dt
            
            # Simple boundary bouncing
            if self.obstacles[i, 0] < 0 or self.obstacles[i, 0] > 5:
                self.obstacles[i, 3] *= -1
            if self.obstacles[i, 1] < 0 or self.obstacles[i, 1] > 5:
                self.obstacles[i, 4] *= -1
    
    def plan_motion(self, target):
        """Solve MPC problem to find optimal controls"""
        # Initial guess (zero controls)
        init_controls = np.zeros(2 * self.horizon)
        
        # Bounds for controls (accel and steer_rate for each step)
        bounds = [(-self.max_accel, self.max_accel) if i % 2 == 0 else 
                 (-self.max_steering_rate, self.max_steering_rate) 
                 for i in range(2 * self.horizon)]
        
        # Optimize control sequence
        result = minimize(self.cost_function, init_controls, args=(target),
                         bounds=bounds, method='SLSQP')
        
        return result.x
    
    def step(self, target):
        """Execute one MPC control step"""
        # Get optimal control sequence
        controls = self.plan_motion(target)
        
        # Apply first control (MPC principle)
        control = controls[:2]
        self.state = self.bicycle_model(self.state, control)
        
        # Update dynamic obstacles
        self.update_obstacles()
        
        return self.state, controls

# Simulation setup
def run_simulation():
    robot = RobotMPC()
    target = np.array([5.0, 5.0])  # Goal position
    
    # Simulation parameters
    sim_steps = 100
    trajectory = []
    planned_paths = []
    
    # Run simulation
    for _ in range(sim_steps):
        current_state, controls = robot.step(target)
        trajectory.append(current_state.copy())
        
        # Store predicted trajectory for visualization
        planned_path = robot.predict_trajectory(current_state, controls)
        planned_paths.append(planned_path)
        
        # Check if reached target
        if np.linalg.norm(current_state[:2] - target) < 0.2:
            print("Target reached!")
            break
    
    # Visualization
    plot_results(trajectory, planned_paths, robot.obstacles, target)

def plot_results(trajectory, planned_paths, obstacles, target):
    plt.figure(figsize=(10, 8))
    
    # Plot actual trajectory
    traj = np.array(trajectory)
    plt.plot(traj[:, 0], traj[:, 1], 'b-', linewidth=2, label='Actual Path')
    plt.quiver(traj[:, 0], traj[:, 1], 
               np.cos(traj[:, 2]), np.sin(traj[:, 2]),
               color='b', scale=15, width=0.003)
    
    # Plot some predicted trajectories
    for i in range(0, len(planned_paths), 5):
        path = planned_paths[i]
        plt.plot(path[:, 0], path[:, 1], 'g--', alpha=0.3, linewidth=1)
    
    # Plot obstacles
    for obs in obstacles:
        circle = Circle(obs[:2], obs[2], color='r', alpha=0.5)
        plt.gca().add_patch(circle)
        # Show obstacle velocity
        plt.arrow(obs[0], obs[1], obs[3], obs[4], 
                 color='r', head_width=0.1, alpha=0.7)
    
    # Plot target
    plt.plot(target[0], target[1], 'g*', markersize=15, label='Target')
    
    plt.xlabel('X position [m]')
    plt.ylabel('Y position [m]')
    plt.title('Robot Motion Planning with MPC')
    plt.legend()
    plt.grid(True)
    plt.axis('equal')
    plt.show()

if __name__ == "__main__":
    run_simulation()
