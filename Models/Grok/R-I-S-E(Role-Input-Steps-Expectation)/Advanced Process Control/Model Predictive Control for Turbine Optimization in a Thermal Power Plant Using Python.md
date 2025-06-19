import numpy as np
from scipy.optimize import minimize

# Turbine Parameters
Ts = 10.0  # Sampling time (s)
T_min, T_max = 400.0, 600.0  # Temperature (°C)
P_min, P_max = 100.0, 200.0  # Pressure (bar)
W_min, W_max = 0.0, 500.0    # Power (MW)
V_min, V_max = 0.0, 1.0      # Valve position (0–1)
F_min, F_max = 0.0, 5.0      # Fuel flow (kg/s)
dV_max = 0.1 * Ts            # Max valve rate (1/s)
dF_max = 0.05 * Ts           # Max fuel rate (kg/s^2)

# 1. Turbine Model
def turbine_dynamics(x, u, d, Ts):
    """
    Discrete-time state-space model: x(k+1) = Ax + Bu + Ed
    x = [T, P, W], u = [V, F], d = [D]
    """
    T, P, W = x
    V, F = u
    D = d[0]
    
    # Parameters
    a_T = -0.05  # Temperature decay
    b_T = 10.0   # Fuel effect on temperature
    c_T = 0.01   # Pressure coupling
    a_P = -0.03  # Pressure decay
    b_P = 5.0    # Valve effect on pressure
    c_P = 0.02   # Fuel coupling
    a_W = -0.1   # Power decay
    b_W = 50.0   # Pressure effect on power
    c_W = -0.1   # Load demand effect
    
    # Nonlinear dynamics (simplified)
    dT = a_T * T + b_T * F + c_T * P
    dP = a_P * P + b_P * V + c_P * F
    dW = a_W * W + b_W * P + c_W * D
    
    x_new = np.array([
        T + Ts * dT,
        P + Ts * dP,
        W + Ts * dW
    ])
    
    return np.clip(x_new, [T_min, P_min, W_min], [T_max, P_max, W_max])

# State-space matrices (linearized for prediction)
A = np.array([[-0.05, 0.01, 0.0],
              [0.0, -0.03, 0.0],
              [0.0, 50.0, -0.1]])
B = np.array([[0.0, 10.0],
              [5.0, 0.02],
              [0.0, 0.0]])
E = np.array([[0.0], [0.0], [-0.1]])
C = np.eye(3)

# 2. MPC Controller
class MPCController:
    def __init__(self, Np=10, Nc=3):
        self.Np = Np  # Prediction horizon
        self.Nc = Nc  # Control horizon
        self.Q = np.diag([5.0, 2.0, 10.0])  # Weights: T, P, W
        self.R = np.diag([0.1, 0.5])        # Weights: V, F (penalize fuel)
        self.S = np.diag([0.01, 0.02])      # Weights: dV, dF

    def predict(self, x0, u_traj, d_traj):
        """
        Predict states over horizon: x(k+i) = A^i*x0 + sum(A^j*Bu + A^j*Ed)
        """
        x_pred = np.zeros((self.Np, 3))
        x = x0.copy()
        for i in range(self.Np):
            u = u_traj[min(i, self.Nc-1)] if i < self.Nc else u_traj[self.Nc-1]
            d = d_traj[i] if i < len(d_traj) else d_traj[-1]
            x = turbine_dynamics(x, u, [d], Ts)
            x_pred[i] = x
        return x_pred

    def cost_function(self, u, x0, ref, d_traj):
        u = np.reshape(u, (self.Nc, 2))
        x_pred = self.predict(x0, u, d_traj)
        J = 0.0
        
        # Tracking error and control effort
        for i in range(self.Np):
            e = x_pred[i] - ref[i]
            J += e.T @ self.Q @ e
            if i < self.Nc:
                J += u[i].T @ self.R @ u[i]
                if i > 0:
                    du = u[i] - u[i-1]
                    J += du.T @ self.S @ du
        
        return J

    def constraints(self, u, x0, u_prev, d_traj):
        u = np.reshape(u, (self.Nc, 2))
        x_pred = self.predict(x0, u, d_traj)
        cons = []
        
        # State constraints
        for i in range(self.Np):
            cons.extend([
                x_pred[i, 0] - T_min,  # T >= T_min
                T_max - x_pred[i, 0],  # T <= T_max
                x_pred[i, 1] - P_min,  # P >= P_min
                P_max - x_pred[i, 1],  # P <= P_max
                x_pred[i, 2] - W_min,  # W >= W_min
                W_max - x_pred[i, 2]   # W <= W_max
            ])
        
        # Input constraints
        for i in range(self.Nc):
            cons.extend([
                u[i, 0] - V_min,  # V >= V_min
                V_max - u[i, 0],  # V <= V_max
                u[i, 1] - F_min,  # F >= F_min
                F_max - u[i, 1]   # F <= F_max
            ])
            
            # Rate constraints
            if i == 0 and u_prev is not None:
                dV = u[i, 0] - u_prev[0]
                dF = u[i, 1] - u_prev[1]
            else:
                dV = u[i, 0] - u[i-1, 0] if i > 0 else 0
                dF = u[i, 1] - u[i-1, 1] if i > 0 else 0
            cons.extend([
                dV_max - dV,  # dV <= dV_max
                dV + dV_max,  # dV >= -dV_max
                dF_max - dF,  # dF <= dF_max
                dF + dF_max   # dF >= -dF_max
            ])
        
        return np.array(cons)

    def compute_control(self, x0, ref, d_traj, u_prev=None):
        u0 = np.tile([0.5, 2.0], (self.Nc, 1)).flatten()
        bounds = [(V_min, V_max), (F_min, F_max)] * self.Nc
        cons = {'type': 'ineq', 'fun': lambda u: self.constraints(u, x0, u_prev, d_traj)}
        
        result = minimize(
            lambda u: self.cost_function(u, x0, ref, d_traj),
            u0,
            method='SLSQP',
            bounds=bounds,
            constraints=cons
        )
        
        if result.success:
            return np.reshape(result.x, (self.Nc, 2))[0]
        return u_prev if u_prev is not None else np.array([0.5, 2.0])

# 3. Simulation Setup
def generate_demand(t_span, scenario):
    D = np.ones(len(t_span)) * 400.0  # Nominal 400 MW
    if scenario == "Step":
        D[100:200] = 450.0  # Step to 450 MW
        D[300:400] = 350.0  # Step to 350 MW
    elif scenario == "Ramp":
        D[100:200] = 400 + np.linspace(0, 50, 100)  # Ramp to 450 MW
        D[300:400] = 400 + np.linspace(0, -50, 100) # Ramp to 350 MW
    elif scenario == "Noisy":
        np.random.seed(42)
        D += np.random.normal(0, 10, len(t_span))  # Noise ±10 MW
    return np.clip(D, 0, 500)

# Simulation
n_sim = 360  # 3600s (1 hour)
t_span = np.arange(0, n_sim * Ts, Ts)
x0 = np.array([450.0, 150.0, 400.0])  # Initial: T=450°C, P=150 bar, W=400 MW
scenarios = ["Step", "Ramp", "Noisy"]

for scenario in scenarios:
    print(f"\nScenario: {scenario}")
    
    # Generate demand (reference for power)
    D_traj = generate_demand(t_span, scenario)
    ref = np.zeros((n_sim, 3))
    ref[:, 0] = 450.0  # Nominal T
    ref[:, 1] = 150.0  # Nominal P
    ref[:, 2] = D_traj  # Track demand
    
    # Initialize
    mpc = MPCController()
    x_traj = np.zeros((n_sim, 3))
    u_traj = np.zeros((n_sim, 2))
    x_traj[0] = x0
    u_traj[0] = [0.5, 2.0]
    
    # Simulate
    for k in range(n_sim-1):
        d_forecast = D_traj[k:min(k+mpc.Np, n_sim)]
        if len(d_forecast) < mpc.Np:
            d_forecast = np.pad(d_forecast, (0, mpc.Np-len(d_forecast)), 'edge')
        
        u = mpc.compute_control(x_traj[k], ref[k:k+mpc.Np], d_forecast, u_traj[k-1] if k > 0 else None)
        u_traj[k+1] = u
        x_traj[k+1] = turbine_dynamics(x_traj[k], u, [D_traj[k]], Ts)
    
    # Metrics
    error = ref - x_traj
    iae = np.sum(np.abs(error), axis=0) * Ts
    energy = np.sum(u_traj[:, 1]) * Ts  # Fuel flow integral (kg)
    violations = np.sum((x_traj[:, 0] < T_min) | (x_traj[:, 0] > T_max) |
                       (x_traj[:, 1] < P_min) | (x_traj[:, 1] > P_max) |
                       (x_traj[:, 2] < W_min) | (x_traj[:, 2] > W_max))
    
    print(f"IAE: T={iae[0]:.2f}°C·s, P={iae[1]:.2f}bar·s, W={iae[2]:.2f}MW·s")
    print(f"Energy Consumption (Fuel): {energy:.2f} kg")
    print(f"Constraint Violations: {violations}")
    print("Sample Outputs (every 600s):")
    print("Time(s) | T(°C) | P(bar) | W(MW) | V | F(kg/s)")
    for k in range(0, n_sim, 60):
        print(f"{t_span[k]:7.1f} | {x_traj[k, 0]:5.2f} | {x_traj[k, 1]:6.2f} | {x_traj[k, 2]:5.2f} | {u_traj[k, 0]:3.2f} | {u_traj[k, 1]:7.2f}")
