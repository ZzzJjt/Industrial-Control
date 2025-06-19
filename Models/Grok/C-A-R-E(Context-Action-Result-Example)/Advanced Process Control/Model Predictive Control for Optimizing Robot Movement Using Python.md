import numpy as np
from scipy.optimize import minimize

# Robot Model
class Robot:
    def __init__(self, dt=1.0):
        self.dt = dt  # Time step (s)
        self.state = np.array([0.0, 1.0, 0.0, 0.0])  # [x, y, vx, vy]
        self.A = np.array([[1, 0, dt, 0],
                          [0, 1, 0, dt],
                          [0, 0, 1, 0],
                          [0, 0, 0, 1]])  # State transition
        self.B = np.array([[0.5*dt**2, 0],
                          [0, 0.5*dt**2],
                          [dt, 0],
                          [0, dt]])  # Input matrix

    def step(self, u):
        """Update state with control input u = [ax, ay]"""
        self.state = self.A @ self.state + self.B @ u
        # Constraints: velocity limits, corridor bounds
        self.state[2] = np.clip(self.state[2], -1.0, 1.0)  # vx
        self.state[3] = np.clip(self.state[3], -1.0, 1.0)  # vy
        self.state[0] = np.clip(self.state[0], 0.0, 10.0)  # x
        self.state[1] = np.clip(self.state[1], 0.0, 2.0)   # y
        return self.state

# Environment (Corridor with Moving Obstacles)
class Environment:
    def __init__(self):
        self.obstacles = [
            {'radius': 0.3, 'x0': 3.0, 'vx': 0.5, 'y0': 1.0, 'vy': 0.0, 'freq': 0.1},
            {'radius': 0.3, 'x0': 6.0, 'vx': -0.5, 'y0': 1.0, 'vy': 0.0, 'freq': 0.1}
        ]
        self.goal = np.array([8.0, 1.0])  # Goal position

    def get_obstacle_positions(self, t):
        """Return obstacle positions at time t"""
        positions = []
        for obs in self.obstacles:
            x = obs['x0'] + obs['vx'] * t
            y = obs['y0'] + 0.5 * np.sin(obs['freq'] * t)
            positions.append([x, y])
        return positions

    def predict_obstacles(self, t, horizon, dt):
        """Predict obstacle positions over horizon"""
        future_positions = []
        for k in range(horizon):
            tk = t + k * dt
            positions = self.get_obstacle_positions(tk)
            future_positions.append(positions)
        return future_positions

# Potential Field Controller (Baseline)
class PotentialField:
    def __init__(self, env):
        self.env = env
        self.k_att = 0.5  # Attractive gain
        self.k_rep = 2.0  # Repulsive gain
        self.d0 = 0.5     # Repulsion distance

    def compute(self, state):
        """Compute control input using potential fields"""
        x, y = state[0], state[1]
        goal = self.env.goal
        obstacles = self.env.get_obstacle_positions(0.0)  # Current time
        
        # Attractive force to goal
        u_att = self.k_att * (goal - np.array([x, y]))
        
        # Repulsive force from obstacles
        u_rep = np.zeros(2)
        for obs in obstacles:
            obs_pos = np.array(obs)
            dist = np.linalg.norm([x, y] - obs_pos)
            if dist < self.d0:
                u_rep += self.k_rep * (1/dist - 1/self.d0) * ([x, y] - obs_pos) / dist**2
        
        # Repulsive force from walls
        if y < self.d0:
            u_rep[1] += self.k_rep * (1/y - 1/self.d0) / y**2
        if y > 2.0 - self.d0:
            u_rep[1] -= self.k_rep * (1/(2.0-y) - 1/self.d0) / (2.0-y)**2
        
        u = u_att - u_rep
        return np.clip(u, -1.0, 1.0)  # Acceleration limits

# MPC Controller
class MPC:
    def __init__(self, robot, env, horizon=10, control_horizon=3):
        self.robot = robot
        self.env = env
        self.horizon = horizon
        self.control_horizon = control_horizon
        self.dt = robot.dt
        self.Q = np.diag([10.0, 10.0, 1.0, 1.0])  # State error weight
        self.R = np.diag([0.1, 0.1])              # Control effort weight
        self.goal = env.goal

    def predict(self, state, u_traj, future_obstacles):
        """Predict states over horizon"""
        states = [state]
        x = state.copy()
        for k in range(self.horizon):
            u_idx = min(k, self.control_horizon - 1)
            u = u_traj[u_idx * 2:(u_idx + 1) * 2]
            x = self.robot.A @ x + self.robot.B @ u
            x[2] = np.clip(x[2], -1.0, 1.0)
            x[3] = np.clip(x[3], -1.0, 1.0)
            x[0] = np.clip(x[0], 0.0, 10.0)
            x[1] = np.clip(x[1], 0.0, 2.0)
            states.append(x.copy())
        return states

    def objective(self, u_flat, state, future_obstacles):
        """MPC objective function"""
        u_traj = np.array(u_flat).reshape(-1, 2)
        states = self.predict(state, u_flat, future_obstacles)
        J = 0.0
        # Cost: state error + control effort
        for k in range(self.horizon + 1):
            state_err = np.array([states[k][0] - self.goal[0], 
                                states[k][1] - self.goal[1], 
                                states[k][2], states[k][3]])
            J += state_err.T @ self.Q @ state_err
            # Obstacle avoidance penalty
            for obs in future_obstacles[k]:
                dist = np.linalg.norm(states[k][:2] - np.array(obs))
                if dist < 0.3:
                    J += 1e5 * (0.3 - dist) ** 2
            # Wall penalty
            if states[k][1] < 0.3:
                J += 1e5 * (0.3 - states[k][1]) ** 2
            if states[k][1] > 1.7:
                J += 1e5 * (states[k][1] - 1.7) ** 2
        for k in range(self.control_horizon):
            J += u_traj[k].T @ self.R @ u_traj[k]
        return J

    def compute(self, state, t):
        """Optimize control actions"""
        future_obstacles = self.env.predict_obstacles(t, self.horizon, self.dt)
        u_init = np.zeros(self.control_horizon * 2)  # Initial guess
        bounds = [(-1.0, 1.0) for _ in range(self.control_horizon * 2)]  # Input limits
        result = minimize(
            fun=lambda u: self.objective(u, state, future_obstacles),
            x0=u_init,
            bounds=bounds,
            method='SLSQP'
        )
        u = result.x[:2]
        return np.clip(u, -1.0, 1.0)

# Simulate Navigation
def simulate(robot, env, mpc, baseline, t_sim=20.0, dt=1.0):
    n_steps = int(t_sim / dt)
    t = np.linspace(0, t_sim, n_steps)
    x_mpc, y_mpc, vx_mpc, vy_mpc, ax_mpc, ay_mpc = [], [], [], [], [], []
    x_base, y_base, vx_base, vy_base, ax_base, ay_base = [], [], [], []
    
    # MPC simulation
    robot_mpc = Robot(dt)
    for i in range(n_steps):
        state = robot_mpc.state
        u = mpc.compute(state, t[i])
        robot_mpc.step(u)
        x_mpc.append(state[0])
        y_mpc.append(state[1])
        vx_mpc.append(state[2])
        vy_mpc.append(state[3])
        ax_mpc.append(u[0])
        ay_mpc.append(u[1])
    
    # Baseline simulation
    robot_base = Robot(dt)
    for i in range(n_steps):
        state = robot_base.state
        u = baseline.compute(state)
        robot_base.step(u)
        x_base.append(state[0])
        y_base.append(state[1])
        vx_base.append(state[2])
        vy_base.append(state[3])
        ax_base.append(u[0])
        ay_base.append(u[1])
    
    # Store results
    results = {
        'time': t,
        'x_mpc': x_mpc, 'y_mpc': y_mpc, 'vx_mpc': vx_mpc, 'vy_mpc': vy_mpc,
        'ax_mpc': ax_mpc, 'ay_mpc': ay_mpc,
        'x_base': x_base, 'y_base': y_base, 'vx_base': vx_base, 'vy_base': vy_base,
        'ax_base': ax_base, 'ay_base': ay_base
    }
    
    # Calculate performance metrics
    goal = env.goal
    mse_mpc = np.mean([(x - goal[0])**2 + (y - goal[1])**2 for x, y in zip(x_mpc, y_mpc)])
    mse_base = np.mean([(x - goal[0])**2 + (y - goal[1])**2 for x, y in zip(x_base, y_base)])
    energy_mpc = np.sum([ax**2 + ay**2 for ax, ay in zip(ax_mpc, ay_mpc)]) * dt
    energy_base = np.sum([ax**2 + ay**2 for ax, ay in zip(ax_base, ay_base)]) * dt
    
    print(f"MPC Position MSE: {mse_mpc:.2f} m²")
    print(f"Baseline Position MSE: {mse_base:.2f} m²")
    print(f"MPC Energy: {energy_mpc:.2f} (m/s²)²·s")
    print(f"Baseline Energy: {energy_base:.2f} (m/s²)²·s")
    
    return results

# Main Execution
if __name__ == "__main__":
    robot = Robot(dt=1.0)
    env = Environment()
    mpc = MPC(robot, env, horizon=10, control_horizon=3)
    baseline = PotentialField(env)
    results = simulate(robot, env, mpc, baseline, t_sim=20.0, dt=1.0)
