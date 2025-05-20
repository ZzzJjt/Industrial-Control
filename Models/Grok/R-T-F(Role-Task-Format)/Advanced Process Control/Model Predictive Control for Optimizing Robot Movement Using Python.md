import numpy as np
from scipy.optimize import minimize

# Step 1: Define Robot and Environment Parameters
ROBOT_RADIUS = 0.2  # Robot radius (m)
V_MAX = 1.0  # Max linear velocity (m/s)
W_MAX = 2.0  # Max angular velocity (rad/s)
DT = 0.1  # Time step (s)
T_SIM = 20.0  # Simulation duration (s)
N = int(T_SIM / DT)  # Number of time steps
NP = 10  # Prediction horizon (steps)
NC = 3   # Control horizon (steps)

# Reference path (straight line to (5, 5))
PATH = np.array([[i * 5.0 / N, i * 5.0 / N] for i in range(N)])

# Obstacles: [x, y, radius]
OBSTACLES = [
    [2.0, 2.0, 0.5],
    [3.5, 3.5, 0.5]
]

# Step 2: Robot Motion Model (Differential Drive Kinematics)
def robot_dynamics(state, u):
    """Kinematic model: state = [x, y, theta], u = [v, w]"""
    x, y, theta = state
    v, w = u
    dx = v * np.cos(theta) * DT
    dy = v * np.sin(theta) * DT
    dtheta = w * DT
    return np.array([x + dx, y + dy, theta + dtheta])

# Step 3: Obstacle Avoidance Cost
def obstacle_cost(state):
    """Penalty for proximity to obstacles"""
    x, y = state[:2]
    cost = 0.0
    for obs in OBSTACLES:
        ox, oy, r = obs
        dist = np.sqrt((x - ox)**2 + (y - oy)**2) - (r + ROBOT_RADIUS)
        if dist < 0.5:  # Activate penalty when close
            cost += 1000.0 / (dist + 0.01)  # High penalty for collision
    return cost

# Step 4: MPC Controller
def mpc_controller(state, ref_path, k):
    """MPC to optimize robot trajectory"""
    # Weights
    Q = np.diag([10.0, 10.0, 1.0])  # State error (x, y, theta)
    R = np.diag([0.1, 0.1])         # Control effort (v, w)
    Q_obst = 1.0                    # Obstacle avoidance weight
    
    # Initial guess for control inputs
    u0 = np.zeros(2 * NC)
    
    # Constraints
    bounds = [(0, V_MAX) if i % 2 == 0 else (-W_MAX, W_MAX) for i in range(2 * NC)]
    
    def objective(u):
        """MPC objective function"""
        cost = 0.0
        x_pred = state.copy()
        
        for i in range(NP):
            u_idx = min(i, NC - 1) * 2
            u_i = u[u_idx:u_idx + 2]
            x_pred = robot_dynamics(x_pred, u_i)
            
            # State error cost
            ref = ref_path[min(k + i, N - 1)]
            e = x_pred[:2] - ref
            cost += e.T @ Q[:2, :2] @ e + (x_pred[2] ** 2) * Q[2, 2]
            
            # Control effort cost
            if i < NC:
                cost += u_i.T @ R @ u_i
                if i > 0:
                    du = u[u_idx:u_idx + 2] - u[u_idx - 2:u_idx]
                    cost += du.T @ R @ du
            
            # Obstacle avoidance cost
            cost += Q_obst * obstacle_cost(x_pred)
        
        return cost
    
    # Optimize
    result = minimize(objective, u0, bounds=bounds, method='SLSQP')
    return result.x[:2]  # Return first control move [v, w]

# Step 5: Simulation Loop
state = np.array([0.0, 0.0, 0.0])  # Initial state [x, y, theta]
state_history = [state]
control_history = []
dist_to_goal = []

# Dynamic obstacle (moving obstacle at t=5s)
dynamic_obstacle = [2.5, 2.5, 0.3]  # [x, y, radius]

for k in range(N):
    # Update dynamic obstacle (moves to [3, 3] after t=5s)
    if k * DT >= 5.0:
        dynamic_obstacle = [3.0, 3.0, 0.3]
    current_obstacles = OBSTACLES + [dynamic_obstacle]
    
    # Compute MPC control
    u = mpc_controller(state, PATH, k)
    u = np.clip(u, [0, -W_MAX], [V_MAX, W_MAX])
    control_history.append(u)
    
    # Update state
    state = robot_dynamics(state, u)
    state_history.append(state)
    
    # Distance to goal
    dist = np.linalg.norm(state[:2] - PATH[-1])
    dist_to_goal.append(dist)
    
    # Log every 1 second
    if k % int(1.0 / DT) == 0:
        print(f"Time: {k * DT:.1f}s | Pos: ({state[0]:.2f}, {state[1]:.2f}) | "
              f"Theta: {state[2]:.2f}rad | Vel: {u[0]:.2f}m/s | AngVel: {u[1]:.2f}rad/s | "
              f"Dist to Goal: {dist:.2f}m")

# Step 6: Performance Metrics
state_history = np.array(state_history)
control_history = np.array(control_history)
error = np.linalg.norm(state_history[:, :2] - PATH, axis=1)
mse = np.mean(error ** 2)
settling_idx = next((i for i, d in enumerate(dist_to_goal) if d < 0.1), N)
settling_time = settling_idx * DT
energy = np.sum(np.linalg.norm(control_history, axis=1) ** 2) * DT  # Proxy for energy

print("\nPerformance Metrics:")
print(f"Mean Squared Error: {mse:.2f} m^2")
print(f"Settling Time: {settling_time:.1f} s")
print(f"Energy Consumption (proxy): {energy:.2f} units")
print(f"Final Distance to Goal: {dist_to_goal[-1]:.2f} m")
