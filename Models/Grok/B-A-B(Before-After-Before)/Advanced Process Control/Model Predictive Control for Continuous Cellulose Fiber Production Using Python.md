import numpy as np
from scipy.optimize import minimize
import matplotlib.pyplot as plt

# Set random seed for reproducibility
np.random.seed(42)

# System parameters
dt = 0.1  # Time step (hours)
N_sim = 200  # Simulation steps (20 hours)
delay = 2.0  # Upstream process delay (hours)
F_nominal = 50.0  # Nominal infeed rate (tons/hour)
V_max = 200.0  # Buffer tank capacity (tons)
V_min = 20.0  # Minimum tank level to avoid underfill (tons)
F_min = 40.0  # Minimum infeed rate (tons/hour)
F_max = 60.0  # Maximum infeed rate (tons/hour)

# Buffer tank dynamics
class BufferTank:
    def __init__(self, V0, delay_steps):
        self.V = V0  # Initial tank level (tons)
        self.delay_steps = delay_steps  # Delay in steps
        self.F_history = [F_nominal] * delay_steps  # Infeed history for delay
        
    def update(self, F_in, D_out):
        # Apply delayed infeed
        F_delayed = self.F_history.pop(0)
        self.F_history.append(F_in)
        # Update tank level
        self.V += (F_delayed - D_out) * dt
        # Enforce tank constraints
        self.V = np.clip(self.V, 0, V_max)
        return self.V

# Demand model (stochastic with diurnal pattern)
def demand(t):
    base = 45.0  # Base demand (tons/hour)
    variation = 10.0 * np.sin(2 * np.pi * t / 24.0)  # Diurnal variation
    noise = np.random.normal(0, 2.0)  # Stochastic noise
    return np.clip(base + variation + noise, 30.0, 60.0)

# MPC Controller
class MPCController:
    def __init__(self, Np, Nc, dt, delay_steps):
        self.Np = Np  # Prediction horizon
        self.Nc = Nc  # Control horizon
        self.dt = dt
        self.delay_steps = delay_steps
        self.Q = 1.0  # Weight on tank level error
        self.R = 0.1  # Weight on control effort
        self.V_ref = 0.9 * V_max  # Target tank level (90% of capacity)
        
    def predict_demand(self, t, Np):
        return np.array([demand(t + i * self.dt) for i in range(Np)])
    
    def cost_function(self, u, V0, D_pred, F_prev, F_history):
        V = V0
        J = 0.0
        F_delayed = F_history.copy()
        
        for i in range(self.Np):
            # Get control input (constant after Nc)
            F = u[min(i, self.Nc-1)] if i < self.Nc else u[-1]
            
            # Apply delayed infeed
            if i >= self.delay_steps:
                F_in = F_delayed[i - self.delay_steps]
            else:
                F_in = F_history[i]
                
            # Update tank level
            V += (F_in - D_pred[i]) * self.dt
            V = np.clip(V, 0, V_max)
            
            # Cost: tracking error + control effort
            J += self.Q * (V - self.V_ref)**2
            if i > 0:
                J += self.R * (F - F_prev)**2
            F_prev = F
            F_delayed.append(F)
            
        return J
    
    def control(self, V0, t, F_prev, F_history):
        D_pred = self.predict_demand(t, self.Np)
        u0 = np.ones(self.Nc) * F_prev  # Initial guess
        bounds = [(F_min, F_max)] * self.Nc
        
        # Optimize control inputs
        result = minimize(
            lambda u: self.cost_function(u, V0, D_pred, F_prev, F_history),
            u0,
            bounds=bounds,
            method='SLSQP'
        )
        
        return result.x[0]  # Return first control action

# Simulation
def run_simulation():
    # Initialize system
    delay_steps = int(delay / dt)
    tank = BufferTank(V0=0.5 * V_max, delay_steps=delay_steps)
    mpc = MPCController(Np=20, Nc=5, dt=dt, delay_steps=delay_steps)
    
    # Storage
    t = np.arange(N_sim) * dt
    V = np.zeros(N_sim)
    F = np.zeros(N_sim)
    D = np.zeros(N_sim)
    V[0] = tank.V
    F[0] = F_nominal
    D[0] = demand(0)
    
    # Main loop
    for i in range(1, N_sim):
        # Current demand
        D[i] = demand(t[i])
        
        # MPC control
        F[i] = mpc.control(V[i-1], t[i], F[i-1], tank.F_history[:-1])
        
        # Update tank
        V[i] = tank.update(F[i], D[i])
    
    # Performance metrics
    underfill_events = np.sum(V < V_min)
    mean_level = np.mean(V)
    mean_control_effort = np.mean(np.abs(np.diff(F)))
    
    # Output results
    print(f"Mean Tank Level: {mean_level:.2f} tons (Target: {mpc.V_ref:.2f} tons)")
    print(f"Underfill Events (V < {V_min:.2f} tons): {underfill_events}")
    print(f"Mean Control Effort: {mean_control_effort:.2f} tons/hour")
    print(f"Constraint Satisfaction: {np.all((F >= F_min) & (F <= F_max))}")
    
    # Visualization (for clarity)
    plt.figure(figsize=(12, 8))
    plt.subplot(3, 1, 1)
    plt.plot(t, V, 'b-', label='Tank Level')
    plt.axhline(mpc.V_ref, color='r', linestyle='--', label='Target Level')
    plt.axhline(V_min, color='k', linestyle='--', label='Min Level')
    plt.xlabel('Time (hours)')
    plt.ylabel('Tank Level (tons)')
    plt.legend()
    plt.grid(True)
    
    plt.subplot(3, 1, 2)
    plt.plot(t, F, 'b-', label='Infeed Rate')
    plt.axhline(F_min, color='k', linestyle='--', label='Min/Max Infeed')
    plt.axhline(F_max, color='k', linestyle='--')
    plt.xlabel('Time (hours)')
    plt.ylabel('Infeed Rate (tons/hour)')
    plt.legend()
    plt.grid(True)
    
    plt.subplot(3, 1, 3)
    plt.plot(t, D, 'b-', label='Demand')
    plt.xlabel('Time (hours)')
    plt.ylabel('Demand (tons/hour)')
    plt.legend()
    plt.grid(True)
    
    plt.tight_layout()
    plt.savefig('cellulose_mpc_results.png')

if __name__ == "__main__":
    run_simulation()
