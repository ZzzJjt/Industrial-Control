import numpy as np
from scipy.optimize import minimize
import matplotlib.pyplot as plt

# Set random seed for reproducibility
np.random.seed(42)

# Robot and Environment Parameters
dt = 0.1  # Time step (s)
N_sim = 100  # Simulation steps (10 s)
v_max = 1.0  # Max speed (m/s)
a_max = 2.0  # Max acceleration (m/s^2)
r_robot = 0.2  # Robot radius (m)
r_safe = 0.5  # Safety distance for obstacles (m)

# Dynamic obstacles (position and velocity)
obstacles = [
    {'p0': np.array([2.0, 1.0]), 'v': np.array([0.2, 0.0])},
    {'p0': np.array([3.0, 3.0]), 'v': np.array([-0.1, -0.2])}
]

# Robot Dynamics
class Robot:
    def __init__(self, x0):
        self.x = x0  # State: [px, py, vx, vy]
        
    def update(self, u):
        # State update: x' = Ax + Bu
        A = np.array([[1, 0, dt, 0],
                      [0, 1, 0, dt],
                      [0, 0, 1, 0],
                      [0, 0, 0, 1]])
        B = np.array([[0, 0],
                      [0, 0],
                      [dt, 0],
                      [0, dt]])
        self.x = A @ self.x + B @ u
        # Enforce speed constraint
        speed = np.sqrt(self.x[2]**2 + self.x[3]**2)
        if speed > v_max:
            self.x[2:4] *= v_max / speed
    
    def get_position(self):
        return self.x[:2]

# Environment Model
class Environment:
    def get_obstacle_positions(self, t):
        positions = []
        for obs in obstacles:
            p = obs['p0'] + obs['v'] * t
            positions.append(p)
        return positions

# MPC Controller
class MPCController:
    def __init__(self, Np, Nc, dt):
        self.Np = Np  # Prediction horizon
        self.Nc = Nc  # Control horizon
        self.dt = dt
        self.Q = np.diag([10.0, 10.0, 1.0, 1.0])  # State error weight
        self.R = np.diag([0.1, 0.1])  # Control effort weight
        self.target = np.array([4.0, 4.0])  # Target position
    
    def predict_obstacles(self, t, Np):
        return [np.array([obs['p0'] + obs['v'] * (t + i * self.dt) for i in range(Np)]) for obs in obstacles]
    
    def cost_function(self, u, x0, t, obstacles):
        u = u.reshape(self.Nc, 2)
        x = x0.copy()
        J = 0.0
        A = np.array([[1, 0, dt, 0],
                      [0, 1, 0, dt],
                      [0, 0, 1, 0],
                      [0, 0, 0, 1]])
        B = np.array([[0, 0],
                      [0, 0],
                      [dt, 0],
                      [0, dt]])
        
        for i in range(self.Np):
            # Control input (constant after Nc)
            u_i = u[min(i, self.Nc-1)] if i < self.Nc else u[-1]
            
            # Update state
            x = A @ x + B @ u_i
            speed = np.sqrt(x[2]**2 + x[3]**2)
            if speed > v_max:
                x[2:4] *= v_max / speed
            
            # Cost: tracking error
            pos_error = x[:2] - self.target
            J += pos_error.T @ self.Q[:2, :2] @ pos_error + x[2:4].T @ self.Q[2:, 2:] @ x[2:4]
            
            # Cost: control effort
            if i > 0:
                J += (u_i - u[max(0, i-1)]).T @ self.R @ (u_i - u[max(0, i-1)])
            
            # Obstacle avoidance penalty
            for obs in obstacles[i]:
                dist = np.linalg.norm(x[:2] - obs)
                if dist < r_safe:
                    J += 1000.0 * (r_safe - dist)**2
        
        return J
    
    def constraints(self, u, x0, t, obstacles):
        u = u.reshape(self.Nc, 2)
        cons = []
        for i in range(self.Nc):
            # Acceleration constraints
            if i > 0:
                a = (u[i] - u[i-1]) / self.dt
                cons.append(a_max**2 - a[0]**2 - a[1]**2)
            else:
                cons.append(a_max**2 - u[i][0]**2 - u[i][1]**2)
        return cons
    
    def control(self, x0, t):
        u0 = np.zeros(self.Nc * 2)  # Initial guess
        bounds = [(-a_max * dt, a_max * dt)] * (self.Nc * 2)
        obs_pred = self.predict_obstacles(t, self.Np)
        
        # Optimize
        result = minimize(
            lambda u: self.cost_function(u, x0, t, obs_pred),
            u0,
            constraints={'type': 'ineq', 'fun': lambda u: self.constraints(u, x0, t, obs_pred)},
            bounds=bounds,
            method='SLSQP'
        )
        
        return result.x[:2]  # Return first control action

# Simulation
def run_simulation():
    # Initialize system
    robot = Robot(np.array([0.0, 0.0, 0.0, 0.0]))
    env = Environment()
    mpc = MPCController(Np=10, Nc=3, dt=dt)
    
    # Storage
    t = np.arange(N_sim) * dt
    path = np.zeros((N_sim, 2))
    u_history = np.zeros((N_sim, 2))
    path[0] = robot.get_position()
    
    # Main loop
    for i in range(1, N_sim):
        u = mpc.control(robot.x, t[i])
        robot.update(u)
        path[i] = robot.get_position()
        u_history[i] = u
    
    # Performance metrics
    final_error = np.linalg.norm(path[-1] - mpc.target)
    smoothness = np.mean(np.linalg.norm(np.diff(u_history, axis=0), axis=1))
    min_dist = np.inf
    for i in range(N_sim):
        for obs in env.get_obstacle_positions(t[i]):
            dist = np.linalg.norm(path[i] - obs)
            min_dist = min(min_dist, dist)
    
    # Output results
    print(f"Final Position Error: {final_error:.2f} m (Target: {mpc.target})")
    print(f"Average Control Smoothness: {smoothness:.2f} m/s^2")
    print(f"Minimum Obstacle Distance: {min_dist:.2f} m (Safe: {r_safe:.2f} m)")
    print(f"Constraints Satisfied: {np.all(np.linalg.norm(u_history, axis=1) <= a_max * dt)}")
    
    # Visualization
    plt.figure(figsize=(8, 8))
    plt.plot(path[:, 0], path[:, 1], 'b-', label='Robot Path')
    plt.plot(mpc.target[0], mpc.target[1], 'r*', label='Target')
    for i, obs in enumerate(obstacles):
        obs_path = np.array([obs['p0'] + obs['v'] * ti for ti in t])
        plt.plot(obs_path[:, 0], obs_path[:, 1], 'k--', label=f'Obstacle {i+1}')
        plt.plot(obs_path[0, 0], obs_path[0, 1], 'ko')
    plt.xlabel('X (m)')
    plt.ylabel('Y (m)')
    plt.legend()
    plt.grid(True)
    plt.axis('equal')
    plt.savefig('robot_mpc_path.png')

if __name__ == "__main__":
    run_simulation()
