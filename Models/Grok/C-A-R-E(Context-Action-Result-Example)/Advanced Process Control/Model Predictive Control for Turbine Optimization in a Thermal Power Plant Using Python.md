import numpy as np
from scipy.optimize import minimize

# Turbine Model
class Turbine:
    def __init__(self, dt=1.0):
        self.dt = dt  # Time step (s)
        self.state = np.array([377.0, 500.0])  # [omega (rad/s), Ts (°C)]
        self.J = 1000.0  # Moment of inertia (kg·m²)
        self.C_T = 500.0  # Thermal capacitance (kJ/°C)
        self.K_f = 0.5  # Fuel-to-temperature gain (°C/kg/s)
        self.K_v = 0.1  # Valve-to-torque gain (N·m/%)
        self.tau_T = 10.0  # Thermal time constant (s)
        self.P_nom = 100.0  # Nominal power (MW)

    def step(self, u, P_d):
        """Update state with control input u = [u_f, u_v] and disturbance P_d"""
        omega, Ts = self.state
        u_f, u_v = u[0], u[1]
        # Nonlinear dynamics
        # d(omega)/dt = (K_v*u_v*Ts - P_d*1e6/omega)/J
        torque = self.K_v * u_v * Ts
        load_torque = P_d * 1e6 / omega if omega != 0 else 0
        domega_dt = (torque - load_torque) / self.J
        # d(Ts)/dt = (K_f*u_f - (Ts - T_amb)/tau_T)/C_T
        T_amb = 25.0  # Ambient temperature (°C)
        dTs_dt = (self.K_f * u_f - (Ts - T_amb) / self.tau_T) / self.C_T
        self.state[0] += domega_dt * self.dt
        self.state[1] += dTs_dt * self.dt
        # Constraints
        self.state[0] = np.clip(self.state[0], 350.0, 400.0)  # omega
        self.state[1] = np.clip(self.state[1], 450.0, 550.0)  # Ts
        return self.state

# PID Controller (Baseline)
class PID:
    def __init__(self, Kp=0.5, Ki=0.01, dt=1.0):
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
        self.prev_error = error
        return np.clip(u, 0, 100)  # General input limit

# MPC Controller
class MPC:
    def __init__(self, turbine, horizon=10, control_horizon=3):
        self.turbine = turbine
        self.horizon = horizon
        self.control_horizon = control_horizon
        self.dt = turbine.dt
        self.Q = np.diag([10.0, 1.0])  # Weight on [omega, Ts] errors
        self.R = np.diag([0.1, 0.01])  # Weight on [u_f, u_v] effort
        self.omega_setpoint = 377.0  # 3600 RPM (rad/s)
        self.Ts_setpoint = 500.0    # °C
        # Linearized model at nominal: omega=377, Ts=500, u_f=5, u_v=50, P_d=100
        self.A = np.array([[1.0, self.turbine.K_v * 50.0 / self.turbine.J * self.dt],
                          [0.0, 1.0 - self.dt / (self.turbine.C_T * self.turbine.tau_T)]])
        self.B = np.array([[0.0, self.turbine.K_v * 500.0 / self.turbine.J * self.dt],
                          [self.turbine.K_f / self.turbine.C_T * self.dt, 0.0]])
        self.C = np.eye(2)

    def predict(self, state, u_traj, P_d_future):
        """Predict states over horizon"""
        states = [state]
        x = state.copy()
        for k in range(self.horizon):
            u_idx = min(k, self.control_horizon - 1)
            u = u_traj[u_idx * 2:(u_idx + 1) * 2]
            # Linearized dynamics: x[k+1] = A*x[k] + B*u[k] + d
            d = np.array([-P_d_future[k] * 1e6 / x[0] / self.turbine.J * self.dt if x[0] != 0 else 0,
                          25.0 / (self.turbine.C_T * self.turbine.tau_T) * self.dt])
            x = self.A @ x + self.B @ u + d
            x[0] = np.clip(x[0], 350.0, 400.0)
            x[1] = np.clip(x[1], 450.0, 550.0)
            states.append(x.copy())
        return states

    def objective(self, u_flat, state, P_d_future, u_prev):
        """MPC objective function"""
        u_traj = np.array(u_flat).reshape(-1, 2)
        states = self.predict(state, u_flat, P_d_future)
        J = 0.0
        # Cost: state error + control effort + rate penalty
        for k in range(self.horizon + 1):
            state_err = np.array([states[k][0] - self.omega_setpoint,
                                 states[k][1] - self.Ts_setpoint])
            J += state_err.T @ self.Q @ state_err
        for k in range(self.control_horizon):
            J += u_traj[k].T @ self.R @ u_traj[k]
            # Rate penalty
            d_f = u_traj[k][0] - (u_prev[0] if k == 0 else u_traj[k-1][0])
            d_v = u_traj[k][1] - (u_prev[1] if k == 0 else u_traj[k-1][1])
            J += 0.1 * (d_f**2 + d_v**2)
        # Constraint penalties
        for k in range(self.horizon + 1):
            if states[k][0] < 350.0:
                J += 1e5 * (350.0 - states[k][0])**2
            if states[k][0] > 400.0:
                J += 1e5 * (states[k][0] - 400.0)**2
            if states[k][1] < 450.0:
                J += 1e5 * (450.0 - states[k][1])**2
            if states[k][1] > 550.0:
                J += 1e5 * (states[k][1] - 550.0)**2
        return J

    def compute(self, state, P_d_future, u_prev):
        """Optimize control actions"""
        u_init = np.array([5.0, 50.0] * self.control_horizon)  # Initial guess
        bounds = [(0, 10)] * self.control_horizon + [(0, 100)] * self.control_horizon
        result = minimize(
            fun=lambda u: self.objective(u, state, P_d_future, u_prev),
            x0=u_init,
            bounds=bounds,
            method='SLSQP'
        )
        u = result.x[:2]
        u[0] = np.clip(u[0], u_prev[0] - 0.5, u_prev[0] + 0.5)  # Rate limits
        u[1] = np.clip(u[1], u_prev[1] - 5.0, u_prev[1] + 5.0)
        return np.clip(u, [0, 0], [10, 100])

# Simulate Turbine Operation
def simulate(turbine, mpc, pid_omega, pid_Ts, t_sim=1000.0, dt=1.0):
    n_steps = int(t_sim / dt)
    t = np.linspace(0, t_sim, n_steps)
    omega_mpc, Ts_mpc, u_f_mpc, u_v_mpc = [], [], [], []
    omega_pid, Ts_pid, u_f_pid, u_v_pid = [], [], [], []
    P_d = [100.0 if ti < 200 else 120.0 for ti in t]  # Load demand step
    
    # MPC simulation
    turbine_mpc = Turbine(dt)
    u_prev = np.array([5.0, 50.0])  # Initial [u_f, u_v]
    for i in range(n_steps):
        state = turbine_mpc.state
        # Forecast P_d (perfect forecast for simplicity)
        P_d_future = P_d[i:min(i + mpc.horizon, n_steps)]
        if len(P_d_future) < mpc.horizon:
            P_d_future = np.pad(P_d_future, (0, mpc.horizon - len(P_d_future)), 
                               mode='edge')
        u = mpc.compute(state, P_d_future, u_prev)
        turbine_mpc.step(u, P_d[i])
        omega_mpc.append(state[0])
        Ts_mpc.append(state[1])
        u_f_mpc.append(u[0])
        u_v_mpc.append(u[1])
        u_prev = u
    
    # PID simulation
    turbine_pid = Turbine(dt)
    for i in range(n_steps):
        state = turbine_pid.state
        u_f = pid_omega.compute(mpc.omega_setpoint, state[0])
        u_v = pid_Ts.compute(mpc.Ts_setpoint, state[1])
        u = np.array([u_f, u_v])
        turbine_pid.step(u, P_d[i])
        omega_pid.append(state[0])
        Ts_pid.append(state[1])
        u_f_pid.append(u_f)
        u_v_pid.append(u_v)
    
    # Calculate performance metrics
    mse_omega_mpc = np.mean((np.array(omega_mpc) - mpc.omega_setpoint)**2)
    mse_Ts_mpc = np.mean((np.array(Ts_mpc) - mpc.Ts_setpoint)**2)
    mse_omega_pid = np.mean((np.array(omega_pid) - mpc.omega_setpoint)**2)
    mse_Ts_pid = np.mean((np.array(Ts_pid) - mpc.Ts_setpoint)**2)
    energy_mpc = np.sum(np.array(u_f_mpc)) * dt / 3600  # Fuel use (kg)
    energy_pid = np.sum(np.array(u_f_pid)) * dt / 3600  # Fuel use (kg)
    
    print(f"MPC Speed MSE: {mse_omega_mpc:.2f} (rad/s)²")
    print(f"MPC Temperature MSE: {mse_Ts_mpc:.2f} °C²")
    print(f"PID Speed MSE: {mse_omega_pid:.2f} (rad/s)²")
    print(f"PID Temperature MSE: {mse_Ts_pid:.2f} °C²")
    print(f"MPC Fuel Use: {energy_mpc:.2f} kg")
    print(f"PID Fuel Use: {energy_pid:.2f} kg")
    
    # Store results
    results = {
        'time': t,
        'omega_mpc': omega_mpc, 'Ts_mpc': Ts_mpc, 'u_f_mpc': u_f_mpc, 'u_v_mpc': u_v_mpc,
        'omega_pid': omega_pid, 'Ts_pid': Ts_pid, 'u_f_pid': u_f_pid, 'u_v_pid': u_v_pid,
        'P_d': P_d
    }
    return results

# Main Execution
if __name__ == "__main__":
    turbine = Turbine(dt=1.0)
    mpc = MPC(turbine, horizon=10, control_horizon=3)
    pid_omega = PID(Kp=0.5, Ki=0.01, dt=1.0)  # Speed PID
    pid_Ts = PID(Kp=0.5, Ki=0.01, dt=1.0)    # Temperature PID
    results = simulate(turbine, mpc, pid_omega, pid_Ts, t_sim=1000.0, dt=1.0)
