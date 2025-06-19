import numpy as np
from scipy.optimize import minimize

# Robot Parameters
Ts = 0.1  # Sampling time (s)
r_robot = 0.2  # Robot radius (m)
v_min, v_max = -1.0, 1.0  # Velocity limits (m/s)
w_min, w_max = -1.0, 1.0  # Angular velocity limits (rad/s)
a_max = 0.5  # Max acceleration (m/s^2)
alpha_max = 1.0  # Max angular acceleration (rad/s^2)
d_safe = 0.3  # Safe distance from obstacles (m)

# 1. Robot Motion Model
def robot_dynamics(x, u, Ts):
    """
    Discrete-time kinematic model: x(k+1) = f(x(k), u(k))
    x = [x, y, theta, v, w], u = [v_cmd, w_cmd]
    """
    x_new = np.zeros(5)
    x_new[0] = x[0] + Ts * x[3] * np.cos(x[2])  # x
    x_new[1] = x[1] + Ts * x[3] * np.sin(x[2])  # y
    x_new[2] = x[2] + Ts * x[4]  # theta
    x_new[3] = u[0]  # v (direct control)
    x_new[4] = u[1]  # w (direct control)
    return x_new

# 2. MPC Controller
class MPCController:
    def __init__(self, Np=10, Nc=5):
        self.Np = Np  # Prediction horizon
        self.Nc = Nc  # Control horizon
        self.Q = np.diag([10, 10, 1, 0.1, 0.1])  # State weights
        self.R = np.diag([0.1, 0.1])  # Input weights
        self.S = np.diag([0.01, 0.01])  # Input rate weights
        self.obstacles = []  # List of [x, y, r, vx, vy]

    def set_obstacles(self, obstacles):
        self.obstacles = obstacles  # [[x, y, r, vx, vy], ...]

    def predict_obstacles(self, t, k):
        """
        Predict obstacle positions over horizon
        """
        obs_pred = []
        for obs in self.obstacles:
            obs_traj = []
            for i in range(self.Np):
                t_i = t + (k + i) * Ts
                x_i = obs[0] + obs[3] * t_i  # x + vx*t
                y_i = obs[1] + obs[4] * t_i  # y + vy*t
                obs_traj.append([x_i, y_i, obs[2]])
            obs_pred.append(obs_traj)
        return obs_pred

    def cost_function(self, u, x0, ref, t, k):
        u = np.reshape(u, (self.Nc, 2))
        x = x0.copy()
        J = 0.0
        obs_pred = self.predict_obstacles(t, k)
        
        # Predict states
        for i in range(self.Np):
            u_i = u[min(i, self.Nc-1)] if i < self.Nc else u[self.Nc-1]
            x = robot_dynamics(x, u_i, Ts)
            
            # Tracking error
            e = x - ref[i]
            J += e.T @ self.Q @ e
            
            # Obstacle avoidance penalty
            for obs in obs_pred[i]:
                d = np.sqrt((x[0] - obs[0])**2 + (x[1] - obs[1])**2) - obs[2] - r_robot
                if d < d_safe:
                    J += 1000 * (d_safe - d)**2
        
        # Control effort and rate
        for i in range(self.Nc):
            J += u[i].T @ self.R @ u[i]
            if i > 0:
                du = u[i] - u[i-1]
                J += du.T @ self.S @ du
        
        return J

    def constraints(self, u, x0, u_prev):
        u = np.reshape(u, (self.Nc, 2))
        cons = []
        
        # Input constraints
        for i in range(self.Nc):
            cons.append(u[i, 0] - v_min)  # v >= v_min
            cons.append(v_max - u[i, 0])  # v <= v_max
            cons.append(u[i, 1] - w_min)  # w >= w_min
            cons.append(w_max - u[i, 1])  # w <= w_max
            
            # Acceleration constraints
            if i == 0 and u_prev is not None:
                dv = (u[i, 0] - u_prev[0]) / Ts
                dw = (u[i, 1] - u_prev[1]) / Ts
            else:
                dv = (u[i, 0] - u[i-1, 0]) / Ts if i > 0 else 0
                dw = (u[i, 1] - u[i-1, 1]) / Ts if i > 0 else 0
            cons.append(dv - a_max)  # dv <= a_max
            cons.append(a_max - dv)  # dv >= -a_max
            cons.append(dw - alpha_max)  # dw <= alpha_max
            cons.append(alpha_max - dw)  # dw >= -alpha_max
        
        return np.array(cons)

    def compute_control(self, x0, ref, t, k, u_prev=None):
        u0 = np.tile([0.5, 0.0], (self.Nc, 1)).flatten()
        bounds = [(v_min, v_max), (w_min, w_max)] * self.Nc
        cons = {'type': 'ineq', 'fun': lambda u: self.constraints(u, x0, u_prev)}
        
        result = minimize(
            lambda u: self.cost_function(u, x0, ref, t, k),
            u0,
            method='SLSQP',
            bounds=bounds,
            constraints=cons
        )
        
        if result.success:
            return np.reshape(result.x, (self.Nc, 2))[0]
        return np.array([0.0, 0.0])

# 3. Simulation Setup
def generate_reference(t_span, scenario):
    ref = np.zeros((len(t_span), 5))
    if scenario == "Straight":
        for i, t in enumerate(t_span):
            ref[i] = [t, 0.0, 0.0, 0.5, 0.0]  # Move along x-axis
    elif scenario == "Curve":
        for i, t in enumerate(t_span):
            x = 2 * np.sin(t / 5)
            y = 2 * np.cos(t / 5)
            theta = np.arctan2(-2 * np.sin(t / 5), 2 * np.cos(t / 5))
            ref[i] = [x, y, theta, 0.5, 0.0]
    elif scenario == "GoalChange":
        for i, t in enumerate(t_span):
            if t < 10:
                ref[i] = [5.0, 0.0, 0.0, 0.5, 0.0]  # Goal at (5,0)
            else:
                ref[i] = [5.0, 5.0, np.pi/4, 0.5, 0.0]  # Goal at (5,5)
    return ref

def generate_obstacles(scenario):
    if scenario == "Static":
        return [[2.0, 1.0, 0.5, 0.0, 0.0], [4.0, -1.0, 0.5, 0.0, 0.0]]  # [x, y, r, vx, vy]
    elif scenario == "Moving":
        return [[2.0, 1.0, 0.5, 0.1, 0.0], [4.0, -1.0, 0.5, -0.1, 0.0]]  # Moving obstacles
    elif scenario == "Mixed":
        return [[2.0, 1.0, 0.5, 0.0, 0.0], [4.0, -1.0, 0.5, 0.1, 0.1]]  # Static + moving
    return []

# 4. Simulation and Evaluation
scenarios = [
    ("Straight", "Static"),
    ("Curve", "Moving"),
    ("GoalChange", "Mixed")
]

for traj_type, obs_type in scenarios:
    print(f"\nScenario: {traj_type} with {obs_type} Obstacles")
    
    # Initialize
    t_span = np.arange(0, 20, Ts)
    n_sim = len(t_span)
    x0 = np.array([0.0, 0.0, 0.0, 0.0, 0.0])  # [x, y, theta, v, w]
    ref = generate_reference(t_span, traj_type)
    obstacles = generate_obstacles(obs_type)
    
    mpc = MPCController()
    mpc.set_obstacles(obstacles)
    
    # Simulate
    x_traj = np.zeros((n_sim, 5))
    u_traj = np.zeros((n_sim, 2))
    x_traj[0] = x0
    
    for k in range(n_sim):
        u = mpc.compute_control(x_traj[k], ref[k:k+mpc.Np], t_span[k], k, u_traj[k-1] if k > 0 else None)
        u_traj[k] = u
        x_traj[k+1] = robot_dynamics(x_traj[k], u, Ts) if k < n_sim-1 else x_traj[k]
    
    # Metrics
    error = x_traj[:, :3] - ref[:, :3]
    iae = np.sum(np.abs(error), axis=0) * Ts
    min_dist = np.inf
    for k in range(n_sim):
        for obs in mpc.predict_obstacles(t_span[k], k):
            d = np.sqrt((x_traj[k, 0] - obs[0][0])**2 + (x_traj[k, 1] - obs[0][1])**2) - obs[0][2] - r_robot
            min_dist = min(min_dist, d)
    
    print(f"IAE: X={iae[0]:.2f}m, Y={iae[1]:.2f}m, Theta={iae[2]:.2f}rad")
    print(f"Minimum Obstacle Distance: {min_dist:.2f}m")
    print("Sample Outputs (every 5s):")
    print("Time(s) | X(m) | Y(m) | Theta(rad) | V(m/s) | W(rad/s)")
    for k in range(0, n_sim, int(5/Ts)):
        print(f"{t_span[k]:7.1f} | {x_traj[k, 0]:4.2f} | {x_traj[k, 1]:4.2f} | {x_traj[k, 2]:9.3f} | {u_traj[k, 0]:6.2f} | {u_traj[k, 1]:8.2f}")
