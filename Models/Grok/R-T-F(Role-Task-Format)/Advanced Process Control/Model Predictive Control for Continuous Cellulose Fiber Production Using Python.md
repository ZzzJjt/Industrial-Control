import numpy as np
from scipy.optimize import minimize

# Step 1: Define System Parameters
TANK_CAPACITY = 1000  # Buffer tank capacity (m^3)
TANK_MIN = 100  # Minimum tank level (m^3)
TANK_TARGET = 900  # Target tank level (m^3)
INF_MAX = 50  # Max infeed rate (m^3/h)
INF_MIN = 0  # Min infeed rate (m^3/h)
DELAY = 2  # Delay between batch stages (h)
BATCH_SIZE = 100  # Volume per batch (m^3)
BATCH_INTERVAL = 0.5  # Time between batch releases (h)
DT = 0.1  # Simulation time step (h)
T_SIM = 24  # Simulation duration (h)
N = int(T_SIM / DT)  # Number of time steps
DELAY_STEPS = int(DELAY / DT)  # Delay in time steps

# Step 2: Simulate Demand (Fluctuating)
t = np.linspace(0, T_SIM, N)
demand = 30 + 10 * np.sin(0.2 * t) + np.random.normal(0, 2, N)  # Demand (m^3/h)
demand = np.clip(demand, 10, 40)  # Bound demand

# Step 3: System Dynamics Model
def system_dynamics(tank_level, infeed_history, demand, k, batch_queue):
    """Simulate tank level dynamics with delayed batch releases."""
    # Process batches (reactor to homogenizer with delay)
    if k % int(BATCH_INTERVAL / DT) == 0 and k < N:
        batch_queue.append(infeed_history[max(0, k - DELAY_STEPS)] * BATCH_INTERVAL)
    
    # Batch release to tank after delay
    inflow = 0
    if k >= DELAY_STEPS and len(batch_queue) > 0:
        inflow = batch_queue.pop(0) / DT  # Convert batch to flow rate
    
    # Tank dynamics: dV/dt = inflow - demand
    dVdt = inflow - demand[k]
    new_level = tank_level + dVdt * DT
    return max(min(new_level, TANK_CAPACITY), TANK_MIN), batch_queue

# Step 4: MPC Controller
def mpc_controller(tank_level, infeed_history, demand_forecast, k, batch_queue):
    """MPC to optimize infeed rate."""
    Np = 20  # Prediction horizon (steps)
    Nc = 5   # Control horizon (steps)
    
    # Weights
    Q = 10   # Tank level error weight
    R = 0.1  # Infeed change weight
    
    # Initial guess for control inputs
    u0 = np.ones(Nc) * infeed_history[-1] if infeed_history else INF_MAX / 2
    
    # Constraints
    bounds = [(INF_MIN, INF_MAX)] * Nc
    
    def objective(u):
        """MPC objective function."""
        cost = 0
        level = tank_level
        queue = batch_queue.copy()
        inf_hist = infeed_history.copy()
        
        for i in range(Np):
            u_idx = min(i, Nc - 1)  # Use last control input beyond Nc
            inf = u[u_idx]
            
            # Simulate system
            if i % int(BATCH_INTERVAL / DT) == 0:
                queue.append(inf * BATCH_INTERVAL)
            inflow = 0
            if i >= DELAY_STEPS and len(queue) > 0:
                inflow = queue.pop(0) / DT
            dVdt = inflow - demand_forecast[min(k + i, N - 1)]
            level = max(min(level + dVdt * DT, TANK_CAPACITY), TANK_MIN)
            
            # Cost: tank level error + infeed change
            cost += Q * (level - TANK_TARGET) ** 2
            if i > 0:
                cost += R * (u[min(i, Nc - 1)] - u[min(i - 1, Nc - 1)]) ** 2
        
        return cost
    
    # Optimize
    result = minimize(objective, u0, bounds=bounds, method='SLSQP')
    return result.x[0]  # Return first control move

# Step 5: Simulation Loop
tank_level = 800  # Initial tank level (m^3)
infeed_history = [30]  # Initial infeed rate (m^3/h)
tank_history = [tank_level]
batch_queue = []

for k in range(N):
    # Demand forecast (perfect for simplicity, could use historical patterns)
    demand_forecast = demand
    
    # Compute MPC control action
    infeed = mpc_controller(tank_level, infeed_history, demand_forecast, k, batch_queue)
    infeed = np.clip(infeed, INF_MIN, INF_MAX)
    infeed_history.append(infeed)
    
    # Update system
    tank_level, batch_queue = system_dynamics(tank_level, infeed_history, demand, k, batch_queue)
    tank_history.append(tank_level)

# Step 6: Output Results
print("Simulation Results:")
print("Time (h) | Tank Level (m^3) | Infeed Rate (m^3/h) | Demand (m^3/h)")
print("-" * 50)
for k in range(0, N, int(1 / DT)):  # Print every 1 hour
    print(f"{t[k]:.1f}     | {tank_history[k]:.1f}         | {infeed_history[k]:.1f}           | {demand[k]:.1f}")

# Performance Metrics
error = np.array(tank_history) - TANK_TARGET
mse = np.mean(error ** 2)
settling_idx = next((i for i, e in enumerate(error[100:]) if abs(e) < 50), N - 100) + 100
settling_time = t[settling_idx]
print("\nPerformance Metrics:")
print(f"Mean Squared Error: {mse:.2f} m^6")
print(f"Settling Time: {settling_time:.1f} h")
print(f"Average Tank Level: {np.mean(tank_history):.1f} m^3")
print(f"Average Infeed Rate: {np.mean(infeed_history):.1f} m^3/h")
