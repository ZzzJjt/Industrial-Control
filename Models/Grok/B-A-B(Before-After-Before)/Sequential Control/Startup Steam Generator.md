<xaiFile name="model.py" contentType="text/python">
import numpy as np

class SteamGeneratorModel:
    def __init__(self, dt=60.0):
        # Model parameters
        self.dt = dt  # Time step (s)
        self.Cv = 1e6  # Heat capacity of drum (J/K)
        self.V = 2.0   # Drum volume (m³)
        self.rho = 1000.0  # Water density (kg/m³)
        self.h_v = 2.8e6  # Latent heat of vaporization (J/kg)
        self.alpha = 0.1  # Heat transfer coefficient
        self.beta = 0.01  # Pressure-flow coupling
        self.Q_nom = 1e7  # Nominal heat input (J/s)
        
        # State: [pressure (bar), temperature (°C), water level (m)]
        self.x0 = np.array([1.0, 20.0, 1.0])
        # Control: [fuel flow (fraction), water feed (kg/s)]
        self.u0 = np.array([0.0, 0.0])

    def dynamics(self, x, u):
        """Non-linear state dynamics: dx/dt = f(x, u)"""
        P, T, L = x  # Pressure (bar), Temperature (°C), Level (m)
        u_f, u_w = u  # Fuel flow (fraction), Water feed (kg/s)
        
        # Heat input from fuel
        Q = u_f * self.Q_nom
        # Steam outflow (non-linear function of pressure)
        m_out = self.beta * P**0.5
        
        # Differential equations
        dP_dt = (self.rho * self.h_v / self.V) * (u_w - m_out)  # Mass balance
        dT_dt = (Q - self.alpha * (T - 20.0)) / self.Cv  # Energy balance
        dL_dt = (u_w - m_out) / (self.rho * self.V)  # Level dynamics
        
        return np.array([dP_dt, dT_dt, dL_dt])

    def step(self, x, u):
        """Discretize dynamics using Euler method: x(k+1) = x(k) + dt * f(x, u)"""
        dx_dt = self.dynamics(x, u)
        x_next = x + self.dt * dx_dt
        return x_next

    def predict(self, x0, u_traj, N):
        """Predict states over horizon N given control trajectory u_traj"""
        x_pred = np.zeros((N+1, 3))
        x_pred[0] = x0
        for k in range(N):
            x_pred[k+1] = self.step(x_pred[k], u_traj[k])
        return x_pred
  </xaiFile>

  <xaiFile name="controller.py" contentType="text/python">
import numpy as np
from scipy.optimize import minimize
from model import SteamGeneratorModel

class NMPC:
    def __init__(self, model, Np=10, Nc=5):
        self.model = model
        self.Np = Np  # Prediction horizon (steps)
        self.Nc = Nc  # Control horizon (steps)
        
        # Weights for objective function
        self.Q = np.diag([100.0, 10.0, 1.0])  # State error weights (P, T, L)
        self.R = np.diag([0.1, 0.1])         # Control effort weights
        self.P_target = 90.0                  # Target pressure (bar)
        self.T_target = 500.0                 # Target temperature (°C)
        self.L_target = 1.0                   # Target level (m)
        
        # Constraints
        self.P_max = 100.0  # Max pressure (bar)
        self.dT_dt_max = 10.0 / 60.0  # Max temp ramp rate (°C/s)
        self.L_min, self.L_max = 0.5, 1.5  # Level bounds (m)
        self.u_min = np.array([0.0, 0.0])  # Min control (fuel, water)
        self.u_max = np.array([1.0, 0.1])  # Max control

    def objective(self, u_flat, x0):
        """Objective function: minimize startup time + constraint penalties"""
        u_traj = u_flat.reshape((self.Nc, 2))
        x_pred = self.model.predict(x0, u_traj, self.Np)
        
        J = 0.0
        for k in range(self.Np + 1):
            x_err = x_pred[k] - np.array([self.P_target, self.T_target, self.L_target])
            J += x_err.T @ self.Q @ x_err
        for k in range(self.Nc):
            J += u_traj[k].T @ self.R @ u_traj[k]
            
        # Soft constraints: penalize violations
        for k in range(self.Np + 1):
            if x_pred[k, 0] > self.P_max:
                J += 1e5 * (x_pred[k, 0] - self.P_max)**2
            if k > 0:
                dT_dt = (x_pred[k, 1] - x_pred[k-1, 1]) / self.model.dt
                if abs(dT_dt) > self.dT_dt_max:
                    J += 1e5 * (abs(dT_dt) - self.dT_dt_max)**2
            if x_pred[k, 2] < self.L_min:
                J += 1e5 * (self.L_min - x_pred[k, 2])**2
            if x_pred[k, 2] > self.L_max:
                J += 1e5 * (x_pred[k, 2] - self.L_max)**2
        
        return J

    def constraints(self, u_flat):
        """Hard constraints on control inputs"""
        u_traj = u_flat.reshape((self.Nc, 2))
        cons = []
        for k in range(self.Nc):
            cons.extend([u_traj[k, i] - self.u_min[i] for i in range(2)])  # u >= u_min
            cons.extend([self.u_max[i] - u_traj[k, i] for i in range(2)])  # u <= u_max
        return np.array(cons)

    def optimize(self, x0):
        """Solve NMPC optimization problem"""
        u_init = np.tile(self.u_max, (self.Nc, 1)).flatten()
        bounds = [(self.u_min[i], self.u_max[i]) for i in range(2) for _ in range(self.Nc)]
        cons = {'type': 'ineq', 'fun': self.constraints}
        
        result = minimize(
            fun=lambda u: self.objective(u, x0),
            x0=u_init,
            bounds=bounds,
            constraints=cons,
            method='SLSQP'
        )
        
        u_opt = result.x.reshape((self.Nc, 2))
        return u_opt[0]  # Apply first control action
  </xaiFile>

  <xaiFile name="main.py" contentType="text/python">
import numpy as np
import matplotlib.pyplot as plt
from model import SteamGeneratorModel
from controller import NMPC

def simulate_pid(model, T_sim=1800):
    """Baseline PID controller for comparison"""
    x = model.x0.copy()
    u = np.zeros(2)
    x_traj = [x]
    u_traj = [u]
    t = np.arange(0, T_sim, model.dt)
    
    # PID parameters
    Kp_P, Ki_P = 0.02, 0.001  # Pressure PID
    Kp_T, Ki_T = 0.01, 0.0005  # Temperature PID
    P_target, T_target = 90.0, 500.0
    I_P, I_T = 0.0, 0.0
    
    for _ in t[:-1]:
        # PID for pressure (controls water feed)
        e_P = P_target - x[0]
        I_P += e_P * model.dt
        u[1] = np.clip(Kp_P * e_P + Ki_P * I_P, 0.0, 0.1)
        
        # PID for temperature (controls fuel flow)
        e_T = T_target - x[1]
        I_T += e_T * model.dt
        u[0] = np.clip(Kp_T * e_T + Ki_T * I_T, 0.0, 1.0)
        
        x = model.step(x, u)
        x_traj.append(x)
        u_traj.append(u.copy())
    
    return np.array(t), np.array(x_traj), np.array(u_traj)

def simulate_nmpc(model, nmpc, T_sim=1800):
    """Simulate NMPC startup"""
    x = model.x0.copy()
    x_traj = [x]
    u_traj = []
    t = np.arange(0, T_sim, model.dt)
    
    for _ in t[:-1]:
        u = nmpc.optimize(x)
        x = model.step(x, u)
        x_traj.append(x)
        u_traj.append(u)
    
    return np.array(t), np.array(x_traj), np.array(u_traj)

def plot_results(t_pid, x_pid, u_pid, t_nmpc, x_nmpc, u_nmpc):
    """Visualize NMPC vs PID performance"""
    plt.figure(figsize=(12, 10))
    
    # Pressure
    plt.subplot(3, 1, 1)
    plt.plot(t_pid, x_pid[:, 0], label='PID')
    plt.plot(t_nmpc, x_nmpc[:, 0], label='NMPC')
    plt.axhline(90.0, color='r', linestyle='--', label='Target')
    plt.axhline(100.0, color='k', linestyle='--', label='Max')
    plt.ylabel('Pressure (bar)')
    plt.legend()
    
    # Temperature
    plt.subplot(3, 1, 2)
    plt.plot(t_pid, x_pid[:, 1], label='PID')
    plt.plot(t_nmpc, x_nmpc[:, 1], label='NMPC')
    plt.axhline(500.0, color='r', linestyle='--', label='Target')
    plt.ylabel('Temperature (°C)')
    plt.legend()
    
    # Water Level
    plt.subplot(3, 1, 3)
    plt.plot(t_pid, x_pid[:, 2], label='PID')
    plt.plot(t_nmpc, x_nmpc[:, 2], label='NMPC')
    plt.axhline(1.0, color='r', linestyle='--', label='Target')
    plt.axhline(0.5, color='k', linestyle='--', label='Min')
    plt.axhline(1.5, color='k', linestyle='--', label='Max')
    plt.ylabel('Water Level (m)')
    plt.xlabel('Time (s)')
    plt.legend()
    
    plt.tight_layout()
    plt.show()

if __name__ == "__main__":
    # Initialize model and NMPC
    model = SteamGeneratorModel(dt=60.0)
    nmpc = NMPC(model, Np=10, Nc=5)
    
    # Simulate PID and NMPC
    T_sim = 1800  # 30 minutes
    t_pid, x_pid, u_pid = simulate_pid(model, T_sim)
    t_nmpc, x_nmpc, u_nmpc = simulate_nmpc(model, nmpc, T_sim)
    
    # Plot results
    plot_results(t_pid, x_pid, u_pid, t_nmpc, x_nmpc, u_nmpc)
    
    # Calculate startup time and energy use
    P_target, T_target = 90.0, 500.0
    pid_startup = next((i for i, x in enumerate(x_pid) if x[0] >= P_target and x[1] >= T_target), len(x_pid))
    nmpc_startup = next((i for i, x in enumerate(x_nmpc) if x[0] >= P_target and x[1] >= T_target), len(x_nmpc))
    pid_energy = np.sum(u_pid[:, 0]) * model.dt * model.Q_nom / 1e6  # MJ
    nmpc_energy = np.sum(u_nmpc[:, 0]) * model.dt * model.Q_nom / 1e6  # MJ
    
    print(f"PID Startup Time: {pid_startup * model.dt / 60:.1f} min")
    print(f"NMPC Startup Time: {nmpc_startup * model.dt / 60:.1f} min")
    print(f"PID Energy Use: {pid_energy:.1f} MJ")
    print(f"NMPC Energy Use: {nmpc_energy:.1f} MJ")
  </xaiFile>
