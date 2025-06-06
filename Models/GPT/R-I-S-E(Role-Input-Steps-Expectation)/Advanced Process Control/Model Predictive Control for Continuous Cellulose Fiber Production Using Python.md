import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize

# Parameters
T_total = 48      # total time horizon in hours
dt = 1            # time step (hour)
N = int(T_total / dt)  # simulation steps
delay = 2         # 2-hour delay
capacity = 1000   # tank volume in m^3
inflow_rate = 50  # tons/hour (constant feed input)
conversion = 1.0  # tons -> m^3 for simplification

# Demand profile (fluctuating)
np.random.seed(42)
demand = 40 + 10 * np.sin(np.linspace(0, 3*np.pi, N)) + np.random.randn(N)

# Initialize state variables
tank_level = np.zeros(N+1)
feed_queue = np.zeros(N + delay + 1)  # inflow is delayed
buffer = []

# Control bounds
u_min, u_max = 0, 60  # feed rate constraints

def simulate_step(level, queue, demand_t, u):
    queue = np.roll(queue, -1)
    queue[-1] = u
    inflow = queue[0] * conversion
    new_level = max(0, min(capacity, level + inflow - demand_t))
    return new_level, queue

def mpc_controller(current_level, queue, demand_forecast, horizon=6):
    def cost(u_seq):
        level = current_level
        q = queue.copy()
        total_penalty = 0
        for i in range(horizon):
            u = np.clip(u_seq[i], u_min, u_max)
            level, q = simulate_step(level, q, demand_forecast[i], u)
            penalty = (capacity - level)**2  # penalize low tank levels
            total_penalty += penalty
        return total_penalty

    u0 = np.ones(horizon) * inflow_rate
    bounds = [(u_min, u_max)] * horizon
    result = minimize(cost, u0, bounds=bounds)
    return result.x[0]

    for t in range(N):
    horizon = min(6, N - t)
    forecast = demand[t:t+horizon]
    u = mpc_controller(tank_level[t], feed_queue, forecast, horizon)
    tank_level[t+1], feed_queue = simulate_step(tank_level[t], feed_queue, demand[t], u)
    buffer.append(u)

# Plot results
plt.figure(figsize=(12, 5))
plt.subplot(1, 2, 1)
plt.plot(tank_level[:-1], label='Tank Level')
plt.axhline(capacity, color='gray', linestyle='--')
plt.title("Tank Level")
plt.xlabel("Hour")
plt.ylabel("m³")
plt.grid()

plt.subplot(1, 2, 2)
plt.plot(demand, label='Demand')
plt.plot(buffer, label='Feed Rate (MPC)')
plt.title("Demand vs MPC Feed Rate")
plt.xlabel("Hour")
plt.ylabel("tons/hr")
plt.legend()
plt.grid()
plt.tight_layout()
plt.show()
