import numpy as np
from scipy.optimize import minimize

# Reactor Model (Tank with Delayed Infeed)
class CelluloseProcess:
    def __init__(self, dt=60.0, delay=7200.0, tank_capacity=1000.0):
        self.dt = dt  # Time step (s)
        self.delay_steps = int(delay / dt)  # Delay in steps (2 hours)
        self.tank_capacity = tank_capacity  # Tank volume (m³)
        self.tank_level = 500.0  # Initial level (m³)
        self.infeed_history = [50.0] * self.delay_steps  # Infeed history (tons/hour)
        self.outfeed = 50.0  # Current outfeed (tons/hour)
        self.density = 1.0  # tons/m³ (1 ton/hour = 1 m³/hour)

    def step(self, infeed, outfeed):
        """Simulate tank dynamics with delayed infeed"""
        self.infeed_history.append(infeed)
        delayed_infeed = self.infeed_history.pop(0)
        self.outfeed = outfeed
        # Tank dynamics: dV/dt = infeed - outfeed (m³/hour)
        dV_dt = (delayed_infeed - outfeed) / 3600.0  # Convert to m³/s
        self.tank_level += dV_dt * self.dt
        self.tank_level = np.clip(self.tank_level, 0, self.tank_capacity)
        return self.tank_level

# PID Controller (Baseline)
class PID:
    def __init__(self, Kp=0.5, Ki=0.001, dt=60.0):
        self.Kp = Kp
        self.Ki = Ki
        self.dt = dt
        self.integral = 0.0
        self.prev_error = 0.0

    def compute(self, setpoint, pv):
        """Compute PID control signal"""
        error = setpoint - pv
        self.integral += error * self.dt
        u = self.Kp * error + self.Ki * self.integral
        return np.clip(u, 0, 75)  # Infeed limit: 0–75 tons/hour

# MPC Controller
class MPC:
    def __init__(self, process, horizon=180, control_horizon=30):
        self.process = process
        self.horizon = horizon  # Prediction horizon (minutes)
        self.control_horizon = control_horizon  # Control horizon (minutes)
        self.dt = process.dt / 60.0  # Time step (minutes)
        self.Q = 1.0  # Weight on level error
        self.R = 0.1  # Weight on control effort
        self.level_setpoint = 500.0  # Tank level setpoint (m³)

    def predict(self, u_traj, level, infeed_history, outfeed_future):
        """Predict tank level over horizon"""
        level_pred = [level]
        infeed_hist = infeed_history.copy()
        for k in range(self.horizon):
            # Use control horizon for u
            u = u_traj[min(k, self.control_horizon - 1)]
            delayed_infeed = infeed_hist.pop(0)
            infeed_hist.append(u)
            # Tank dynamics
            dV_dt = (delayed_infeed - outfeed_future[k]) / 60.0  # m³/min
            level_next = level_pred[-1] + dV_dt * self.dt
            level_next = np.clip(level_next, 0, self.process.tank_capacity)
            level_pred.append(level_next)
        return level_pred

    def objective(self, u_flat, level, infeed_history, outfeed_future):
        """MPC objective function"""
        u_traj = u_flat[:self.control_horizon]
        level_pred = self.predict(u_traj, level, infeed_history, outfeed_future)
        # Cost: level error + control effort
        J = 0.0
        for k in range(self.horizon + 1):
            J += self.Q * (level_pred[k] - self.level_setpoint) ** 2
        for k in range(self.control_horizon):
            J += self.R * u_traj[k] ** 2
        # Penalty for constraint violations
        for k in range(self.horizon + 1):
            if level_pred[k] < 100:
                J += 1e5 * (100 - level_pred[k]) ** 2
            if level_pred[k] > 900:
                J += 1e5 * (level_pred[k] - 900) ** 2
        return J

    def compute(self, level, infeed_history, outfeed_future):
        """Optimize control actions"""
        u_init = np.ones(self.control_horizon) * 50.0  # Initial guess
        bounds = [(0, 75) for _ in range(self.control_horizon)]  # Infeed limits
        result = minimize(
            fun=lambda u: self.objective(u, level, infeed_history, outfeed_future),
            x0=u_init,
            bounds=bounds,
            method='SLSQP'
        )
        return np.clip(result.x[0], 0, 75)  # Return first control action

# Simulate Production System
def simulate(process, mpc, pid, t_sim=14400, dt=60.0):
    """Simulate MPC and PID control with demand increase"""
    t = np.arange(0, t_sim, dt)
    n_steps = len(t)
    tank_level_mpc = []
    infeed_mpc = []
    tank_level_pid = []
    infeed_pid = []
    outfeed = []
    
    # Demand profile
    outfeed_traj = [50.0 if ti < 7200 else 60.0 for ti in t]  # Step at t=7200s
    
    # MPC simulation
    process_mpc = CelluloseProcess(dt=dt)
    for i in range(n_steps):
        # Forecast outfeed (perfect forecast for simplicity)
        outfeed_future = outfeed_traj[i:min(i + int(mpc.horizon * 60 / dt), n_steps)]
        if len(outfeed_future) < mpc.horizon:
            outfeed_future = np.pad(outfeed_future, (0, mpc.horizon - len(outfeed_future)), 
                                   mode='edge')
        # Compute MPC control
        u = mpc.compute(process_mpc.tank_level, process_mpc.infeed_history, outfeed_future)
        level = process_mpc.step(u, outfeed_traj[i])
        tank_level_mpc.append(level)
        infeed_mpc.append(u)
        outfeed.append(outfeed_traj[i])
    
    # PID simulation
    process_pid = CelluloseProcess(dt=dt)
    for i in range(n_steps):
        u = pid.compute(mpc.level_setpoint, process_pid.tank_level)
        level = process_pid.step(u, outfeed_traj[i])
        tank_level_pid.append(level)
        infeed_pid.append(u)
    
    return t, tank_level_mpc, infeed_mpc, tank_level_pid, infeed_pid, outfeed

# Main Execution
if __name__ == "__main__":
    # Initialize components
    dt = 60.0  # Time step (s)
    process = CelluloseProcess(dt=dt, delay=7200.0, tank_capacity=1000.0)
    mpc = MPC(process, horizon=180, control_horizon=30)  # 180 min horizon
    pid = PID(Kp=0.5, Ki=0.001, dt=dt)
    
    # Simulate
    t, tank_level_mpc, infeed_mpc, tank_level_pid, infeed_pid, outfeed = \
        simulate(process, mpc, pid, t_sim=14400, dt=dt)
    
    # Store results for analysis (no plotting)
    results = {
        'time': t,
        'tank_level_mpc': tank_level_mpc,
        'infeed_mpc': infeed_mpc,
        'tank_level_pid': tank_level_pid,
        'infeed_pid': infeed_pid,
        'outfeed': outfeed
    }
    
    # Calculate performance metrics
    setpoint = 500.0
    mse_mpc = np.mean((np.array(tank_level_mpc) - setpoint) ** 2)
    mse_pid = np.mean((np.array(tank_level_pid) - setpoint) ** 2)
    print(f"MPC Tank Level MSE: {mse_mpc:.2f} m³²")
    print(f"PID Tank Level MSE: {mse_pid:.2f} m³²")
