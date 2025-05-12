import numpy as np
from scipy.optimize import minimize
import matplotlib.pyplot as plt

# Set random seed for reproducibility
np.random.seed(42)

# Turbine System Parameters
dt = 0.1  # Time step (hours)
N_sim = 200  # Simulation steps (20 hours)
T0 = 500.0  # Initial steam temperature (C)
P0 = 10.0  # Initial power output (MW)
m_max = 50.0  # Max steam flow rate (kg/s)
m_min = 10.0  # Min steam flow rate (kg/s)
P_max = 15.0  # Max power output (MW)
P_min = 5.0  # Min power output (MW)
T_max = 550.0  # Max steam temperature (C)
T_min = 450.0  # Min steam temperature (C)
C_th = 1000.0  # Thermal capacitance (kJ/C)
R_th = 0.05  # Thermal resistance (C/kW)
eta = 0.85  # Turbine efficiency

# Turbine Dynamics
class Turbine:
    def __init__(self, T0, P0):
        self.T = T0  # Steam temperature (C)
        self.P = P0  # Power output (MW)
        
    def update(self, m, T_amb, P_load):
        # Thermal dynamics
        Q = m * eta * (self.T - T_amb) / 1000.0  # Heat to power (MW)
        dTdt = (m * (T_amb + 1000.0) - Q * 1000.0) / C_th - (self.T - T_amb) / (R_th * C_th)
        self.T += dTdt * dt
        
        # Power dynamics (simplified)
        self.P = Q * eta
        self.P = np.clip(self.P, P_min, P_max)
        self.T = np.clip(self.T, T_min, T_max)
        
        return self.P, self.T

# Environmental and Load Model
class Environment:
    def __init__(self):
        self.rng = np.random.default_rng(42)
    
    def get_conditions(self, t):
        # Ambient temperature: diurnal variation + noise
        T_amb = 25.0 + 5.0 * np.sin(2 * np.pi * t / 24.0) + self.rng.normal(0, 1.0)
        # Load demand: step changes with noise
        P_load = 12.0 if t < 8.0 or t > 16.0 else 8.0
        P_load += self.rng.normal(0, 0.5)
        return T_amb, np.clip(P_load, P_min, P_max)

# MPC Controller
class MPCController:
    def __init__(self, Np, Nc, dt):
        self.Np = Np  # Prediction horizon
        self.Nc = Nc  # Control horizon
        self.dt = dt
        self.Q = np.array([[10.0, 0.0], [0.0, 1.0]])  # Weight: [power, temperature]
        self.R = 0.1  # Control effort weight
        self.P_ref = 10.0  # Nominal power reference (MW)
        
    def predict_load(self, t, Np, env):
        return np.array([env.get_conditions(t + i * self.dt)[1] for i in range(Np)])
    
    def cost_function(self, u, x0, t, env):
        u = u.reshape(self.Nc, 1)
        T, P = x0
        J = 0.0
        T_amb = env.get_conditions(t)[0]
        P_load = self.predict_load(t, self.Np, env)
        
        for i in range(self.Np):
            m = u[min(i, self.Nc-1)] if i < self.Nc else u[-1]
            Q = m * eta * (T - T_amb) / 1000.0
            dTdt = (m * (T_amb + 1000.0) - Q * 1000.0) / C_th - (T - T_amb) / (R_th * C_th)
            T += dTdt * self.dt
            P = Q * eta
            P = np.clip(P, P_min, P_max)
            T = np.clip(T, T_min, T_max)
            
            # Cost: tracking error + temperature stability
            error = np.array([P - P_load[i], T - 500.0])
            J += error.T @ self.Q @ error
            
            # Control effort
            if i > 0:
                J += self.R * (m - u[max(0, i-1)])**2
        
        return J
    
    def constraints(self, u):
        u = u.reshape(self.Nc, 1)
        cons = []
        for i in range(self.Nc):
            cons.append(m_max - u[i])  # Upper bound
            cons.append(u[i] - m_min)  # Lower bound
        return cons
    
    def control(self, x0, t, env):
        u0 = np.ones(self.Nc) * 30.0  # Initial guess
        bounds = [(m_min, m_max)] * self.Nc
        
        result = minimize(
            lambda u: self.cost_function(u, x0, t, env),
            u0,
            constraints={'type': 'ineq', 'fun': self.constraints},
            bounds=bounds,
            method='SLSQP'
        )
        
        return result.x[0]  # First control action

# Simulation
def run_simulation():
    # Initialize system
    turbine = Turbine(T0, P0)
    env = Environment()
    mpc = MPCController(Np=10, Nc=3, dt=dt)
    
    # Storage
    t = np.arange(N_sim) * dt
    P = np.zeros(N_sim)
    T = np.zeros(N_sim)
    m = np.zeros(N_sim)
    P_load = np.zeros(N_sim)
    P[0] = P0
    T[0] = T0
    m[0] = 30.0
    P_load[0] = env.get_conditions(0)[1]
    
    # Main loop
    for i in range(1, N_sim):
        T_amb, P_load[i] = env.get_conditions(t[i])
        m[i] = mpc.control([T[i-1], P[i-1]], t[i], env)
        P[i], T[i] = turbine.update(m[i], T_amb, P_load[i])
    
    # Performance metrics
    power_error = np.mean(np.abs(P - P_load))
    energy_efficiency = np.mean(P / (m + 1e-6) * eta)  # Avoid division by zero
    constraint_violations = np.sum((m < m_min) | (m > m_max) | (P < P_min) | (P > P_max) | (T < T_min) | (T > T_max))
    
    # Output results
    print(f"Mean Power Tracking Error: {power_error:.2f} MW")
    print(f"Average Energy Efficiency: {energy_efficiency:.2f} MW/(kg/s)")
    print(f"Constraint Violations: {constraint_violations}")
    print(f"Final Power Output: {P[-1]:.2f} MW (Target: {P_load[-1]:.2f} MW)")
    print(f"Final Steam Temperature: {T[-1]:.2f} C")
    
    # Visualization
    plt.figure(figsize=(12, 8))
    plt.subplot(3, 1, 1)
    plt.plot(t, P, 'b-', label='Power Output')
    plt.plot(t, P_load, 'r--', label='Load Demand')
    plt.xlabel('Time (hours)')
    plt.ylabel('Power (MW)')
    plt.legend()
    plt.grid(True)
    
    plt.subplot(3, 1, 2)
    plt.plot(t, T, 'b-', label='Steam Temperature')
    plt.axhline(T_min, color='k', linestyle='--', label='Min/Max Temp')
    plt.axhline(T_max, color='k', linestyle='--')
    plt.xlabel('Time (hours)')
    plt.ylabel('Temperature (C)')
    plt.legend()
    plt.grid(True)
    
    plt.subplot(3, 1, 3)
    plt.plot(t, m, 'b-', label='Steam Flow Rate')
    plt.axhline(m_min, color='k', linestyle='--', label='Min/Max Flow')
    plt.axhline(m_max, color='k', linestyle='--')
    plt.xlabel('Time (hours)')
    plt.ylabel('Flow Rate (kg/s)')
    plt.legend()
    plt.grid(True)
    
    plt.tight_layout()
    plt.savefig('turbine_mpc_results.png')

if __name__ == "__main__":
    run_simulation()
