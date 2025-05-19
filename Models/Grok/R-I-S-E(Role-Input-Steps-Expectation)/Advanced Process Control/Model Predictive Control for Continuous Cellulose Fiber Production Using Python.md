import numpy as np
from scipy.optimize import minimize

# System Parameters
Ts = 0.1  # Sampling time (hours)
tau = 2   # Delay (hours)
n_tau = int(tau / Ts)  # Delay steps
tank_capacity = 1000  # Tank capacity (tons)
F_nominal = 50  # Nominal feed rate (tons/hour)
D_mean = 50  # Mean demand (tons/hour)
D_std = 5    # Demand standard deviation
L_min, L_max = 100, 900  # Tank level constraints (tons)
F_min, F_max = 40, 60    # Feed rate constraints (tons/hour)

# 2. System Model
def simulate_system(L0, F_traj, D_traj, Ts, n_tau, n_sim):
    """
    Simulate tank level dynamics: L(k+1) = L(k) + Ts*(F(k-n_tau) - D(k))
    """
    L = np.zeros(n_sim)
    L[0] = L0
    for k in range(1, n_sim):
        F_delayed = F_traj[max(0, k - n_tau)] if k >= n_tau else 0
        L[k] = L[k-1] + Ts * (F_delayed - D_traj[k-1])
        L[k] = np.clip(L[k], 0, tank_capacity)
    return L

# Generate Stochastic Demand
def generate_demand(n_sim, mean=D_mean, std=D_std):
    np.random.seed(42)
    D = np.random.normal(mean, std, n_sim)
    # Add demand spike at t=50–70 hours
    D[int(50/Ts):int(70/Ts)] += 10
    return np.clip(D, 0, 100)

# 3. MPC Controller
def mpc_controller(L_current, F_history, D_forecast, setpoint, Np=20, Nc=5):
    """
    MPC to optimize feed rate F, accounting for delay and demand.
    Np: Prediction horizon, Nc: Control horizon
    """
    # Weights
    Q = 10  # Level tracking weight
    R = 0.1 # Control effort weight
    S = 0.01 # Control rate weight

    # Prediction matrices (state: L, input: F, disturbance: D)
    def predict(L, F, D, Np):
        L_pred = np.zeros(Np)
        L_pred[0] = L
        for i in range(1, Np):
            k_delay = max(0, i - n_tau)
            F_delayed = F[k_delay] if k_delay < len(F) else F[-1]
            L_pred[i] = L_pred[i-1] + Ts * (F_delayed - D[i-1])
        return L_pred

    # Objective function
    def objective(u):
        u = np.reshape(u, (Nc, 1))
        F = np.concatenate([F_history[-n_tau:], u, u[-1:]*np.ones((Np-Nc-n_tau,1))]) if len(F_history) >= n_tau else np.pad(F_history, (0, Np-len(F_history)), 'edge')
        L_pred = predict(L_current, F, D_forecast)
        error = L_pred - setpoint
        J = Q * np.sum(error**2) + R * np.sum(u**2)
        if len(u) > 1:
            J += S * np.sum((u[1:] - u[:-1])**2)
        return J

    # Constraints
    def constraint(u):
        u = np.reshape(u, (Nc, 1))
        F = np.concatenate([F_history[-n_tau:], u, u[-1:]*np.ones((Np-Nc-n_tau,1))]) if len(F_history) >= n_tau else np.pad(F_history, (0, Np-len(F_history)), 'edge')
        L_pred = predict(L_current, F, D_forecast)
        return np.concatenate([
            L_pred - L_min,  # L >= L_min
            L_max - L_pred,  # L <= L_max
            u - F_min,       # u >= F_min
            F_max - u        # u <= F_max
        ])

    # Optimization
    u0 = np.ones(Nc) * F_nominal
    bounds = [(F_min, F_max)] * Nc
    cons = {'type': 'ineq', 'fun': constraint}
    result = minimize(objective, u0, method='SLSQP', bounds=bounds, constraints=cons)
    
    if result.success:
        return result.x[0]
    else:
        return F_history[-1] if F_history.size > 0 else F_nominal

# 4. Simulation
n_sim = int(100 / Ts)  # Simulate 100 hours
L0 = 500  # Initial tank level (tons)
setpoint = 800  # Target level (tons, high to avoid shutdowns)

# Demand scenarios
D_normal = generate_demand(n_sim)
D_high = generate_demand(n_sim, mean=55, std=7)  # Higher mean and variability
D_low = generate_demand(n_sim, mean=45, std=3)   # Lower mean and variability
scenarios = {'Normal': D_normal, 'High': D_high, 'Low': D_low}

# Run simulations
results = {}
for name, D_traj in scenarios.items():
    L = np.zeros(n_sim)
    F = np.zeros(n_sim)
    L[0] = L0
    F_history = np.array([F_nominal] * n_tau)
    
    for k in range(n_sim):
        # Forecast demand (assume perfect forecast for simplicity)
        D_forecast = D_traj[k:min(k+Np, n_sim)]
        if len(D_forecast) < Np:
            D_forecast = np.pad(D_forecast, (0, Np-len(D_forecast)), 'edge')
        
        # Compute control action
        F[k] = mpc_controller(L[k], F_history, D_forecast, setpoint)
        F_history = np.append(F_history[1:], F[k])
        
        # Update state
        L = simulate_system(L0, F, D_traj, Ts, n_tau, n_sim)
    
    results[name] = {'L': L, 'F': F, 'D': D_traj}

# Performance Metrics
def calculate_metrics(L, setpoint, D, Ts):
    error = setpoint - L
    iae = np.sum(np.abs(error)) * Ts  # Integral Absolute Error
    violations = np.sum((L < L_min) | (L > L_max))  # Constraint violations
    throughput = np.sum(D) * Ts  # Total output (tons)
    return iae, violations, throughput

# Output Results
print("MPC Performance Results:")
for name, res in results.items():
    iae, violations, throughput = calculate_metrics(res['L'], setpoint, res['D'], Ts)
    print(f"\nScenario: {name}")
    print(f"IAE: {iae:.2f} ton·h")
    print(f"Constraint Violations: {violations}")
    print(f"Throughput: {throughput:.2f} tons")
    print("Sample Outputs (every 10 hours):")
    print("Time (h) | Level (tons) | Feed Rate (tons/h) | Demand (tons/h)")
    for k in range(0, n_sim, int(10/Ts)):
        print(f"{k*Ts:8.1f} | {res['L'][k]:11.2f} | {res['F'][k]:17.2f} | {res['D'][k]:14.2f}")
