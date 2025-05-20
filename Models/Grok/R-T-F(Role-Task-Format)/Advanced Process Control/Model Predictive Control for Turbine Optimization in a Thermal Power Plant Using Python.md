import numpy as np
from scipy.optimize import minimize
from scipy.integrate import odeint

# Step 1: Define Turbine System Parameters
TURBINE_PARAMS = {
    'C_th': 1e6,  # Thermal capacitance (J/K)
    'R_th': 0.05, # Thermal resistance (K/W)
    'eta': 0.85,  # Turbine efficiency
    'P_nom': 100e6, # Nominal power output (W)
    'T_sp': 550.0, # Temperature setpoint (K)
    'P_max': 120e6, # Max power output (W)
    'P_min': 50e6,  # Min power output (W)
    'Q_max': 150e6, # Max heat input (W)
    'Q_min': 0.0,   # Min heat input (W)
    'T_max': 600.0, # Max temperature (K)
    'T_min': 500.0, # Min temperature (K)
    'dt': 10.0,     # Time step (s)
    'T_sim': 3600.0 # Simulation duration (s)
}

# Step 2: Turbine Dynamic Model
def turbine_dynamics(state, t, Q, T_amb, params):
    """Model turbine temperature dynamics: dT/dt = (Q - P_out - (T - T_amb)/R_th) / C_th"""
    T = state[0]  # Turbine temperature (K)
    C_th = params['C_th']
    R_th = params['R_th']
    eta = params['eta']
    P_out = Q * eta  # Power output proportional to heat input
    dTdt = (Q - P_out - (T - T_amb) / R_th) / C_th
    return [dTdt]

# Linearized state-space model for MPC
def get_state_space(params):
    """Discretized state-space model around nominal conditions"""
    dt = params['dt']
    C_th = params['C_th']
    R_th = params['R_th']
    eta = params['eta']
    
    A = 1.0 - dt / (C_th * R_th)
    B = dt * (1 - eta) / C_th
    C = 1.0
    return A, B, C

# Step 3: Simulate Load Demand and Disturbances
def generate_load_demand(N):
    """Generate varying load demand and ambient temperature"""
    t = np.linspace(0, TURBINE_PARAMS['T_sim'], N)
    # Load demand (W) with daily variation and noise
    load = 80e6 + 20e6 * np.sin(2 * np.pi * t / 3600) + np.random.normal(0, 2e6, N)
    load = np.clip(load, TURBINE_PARAMS['P_min'], TURBINE_PARAMS['P_max'])
    # Ambient temperature (K) with diurnal variation
    T_amb = 300.0 + 10.0 * np.sin(2 * np.pi * t / 3600)
    return load, T_amb

# Step 4: MPC Controller
def mpc_controller(T_current, Q_prev, load_forecast, T_amb_forecast, k, params):
    """MPC to optimize heat input (Q)"""
    Np = 10  # Prediction horizon
    Nc = 3   # Control horizon
    Q_weight = 10.0  # Temperature error weight
    R_weight = 0.1   # Control effort weight
    
    A, B, C = get_state_space(params)
    
    # Initial guess for control inputs
    u0 = np.ones(Nc) * Q_prev
    
    # Constraints
    bounds = [(params['Q_min'], params['Q_max'])] * Nc
    
    def objective(u):
        """MPC objective function"""
        cost = 0.0
        T_pred = T_current
        
        for i in range(Np):
            u_idx = min(i, Nc - 1)
            Q = u[u_idx]
            P_out = Q * params['eta']
            T_amb = T_amb_forecast[min(k + i, len(T_amb_forecast) - 1)]
            T_pred = A * T_pred + B * Q + (params['dt'] / (params['C_th'] * params['R_th'])) * T_amb
            
            # Ensure power meets load demand
            load = load_forecast[min(k + i, len(load_forecast) - 1)]
            power_error = (P_out - load) / params['P_nom']
            
            # Cost: temperature error + power mismatch + control effort
            T_error = T_pred - params['T_sp']
            cost += Q_weight * T_error ** 2 + 1.0 * power_error ** 2
            if i > 0:
                cost += R_weight * (u[min(i, Nc - 1)] - u[min(i - 1, Nc - 1)]) ** 2
        
        return cost
    
    # Optimize
    result = minimize(objective, u0, bounds=bounds, method='SLSQP')
    return result.x[0]  # First control move

# Step 5: Simulation Loop
def simulate_turbine():
    params = TURBINE_PARAMS
    N = int(params['T_sim'] / params['dt'])
    T_history = np.zeros(N)
    Q_history = np.zeros(N)
    P_history = np.zeros(N)
    
    # Initial conditions
    T_history[0] = params['T_sp']
    Q_history[0] = params['P_nom'] / params['eta']
    P_history[0] = Q_history[0] * params['eta']
    
    # Generate load and ambient temperature
    load_demand, T_amb = generate_load_demand(N)
    
    # Simulation
    for k in range(N - 1):
        # Current state
        T_current = T_history[k]
        Q_prev = Q_history[k]
        
        # MPC control
        Q = mpc_controller(T_current, Q_prev, load_demand, T_amb, k, params)
        Q = np.clip(Q, params['Q_min'], params['Q_max'])
        Q_history[k + 1] = Q
        P_history[k + 1] = Q * params['eta']
        
        # Update state using nonlinear model
        t_span = [0, params['dt']]
        state = odeint(turbine_dynamics, [T_current], t_span, args=(Q, T_amb[k], params))
        T_next = state[-1, 0]
        T_next = np.clip(T_next, params['T_min'], params['T_max'])
        T_history[k + 1] = T_next
    
    return T_history, Q_history, P_history, load_demand, T_amb

# Step 6: Run Simulation and Output Results
T_history, Q_history, P_history, load_demand, T_amb = simulate_turbine()

print("Simulation Results:")
print("Time (h) | Temp (K) | Heat Input (MW) | Power Output (MW) | Load Demand (MW) | Amb Temp (K)")
print("-" * 80)
for k in range(0, len(T_history), int(3600 / TURBINE_PARAMS['dt'])):  # Every hour
    t_h = k * TURBINE_PARAMS['dt'] / 3600.0
    print(f"{t_h:.1f}     | {T_history[k]:.1f}   | {Q_history[k]/1e6:.1f}        | "
          f"{P_history[k]/1e6:.1f}         | {load_demand[k]/1e6:.1f}        | {T_amb[k]:.1f}")

# Step 7: Performance Metrics
error_T = T_history - TURBINE_PARAMS['T_sp']
error_P = P_history - load_demand
mse_T = np.mean(error_T ** 2)
mse_P = np.mean(error_P ** 2)
settling_idx = next((i for i, e in enumerate(error_T[100:]) if abs(e) < 5.0), len(T_history) - 100) + 100
settling_time = settling_idx * TURBINE_PARAMS['dt'] / 3600.0
energy_efficiency = np.mean(P_history / Q_history) * 100  # Percentage

print("\nPerformance Metrics:")
print(f"Mean Squared Error (Temperature): {mse_T:.2f} K^2")
print(f"Mean Squared Error (Power): {mse_P/1e12:.2f} MW^2")
print(f"Settling Time: {settling_time:.1f} h")
print(f"Average Energy Efficiency: {energy_efficiency:.1f}%")
print(f"Average Power Output: {np.mean(P_history)/1e6:.1f} MW")
