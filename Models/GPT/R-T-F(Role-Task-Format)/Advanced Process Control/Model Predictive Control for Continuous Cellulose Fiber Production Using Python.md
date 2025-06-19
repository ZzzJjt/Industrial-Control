import numpy as np
from scipy.optimize import minimize

# === SYSTEM PARAMETERS ===
dt = 1.0                     # Time step (hour)
T_total = 24                 # Total simulation time (hours)
N_sim = int(T_total / dt)    # Number of steps
N_pred = 6                   # MPC horizon (6 hours)
delay_hours = 2              # Delay from reactor to tank
delay_steps = int(delay_hours / dt)

tank_volume = 1000           # Max tank volume in m^3
tank_min = 200               # Min safe level
tank_max = 950               # Max safe level

# === DYNAMIC VARIABLES ===
demand_profile = 250 + 50*np.sin(np.linspace(0, 4*np.pi, N_sim))  # Fluctuating demand

feed_delay_buffer = [0.0] * delay_steps  # Queue for delayed feed
tank_level = 500.0                       # Initial tank level
tank_log = [tank_level]
feed_log = []

# === MPC COST FUNCTION ===
def mpc_cost(u_seq, tank_now, delay_buf, demand_seq):
    tank = tank_now
    cost = 0.0
    buf = delay_buf.copy()

    for t in range(len(u_seq)):
        buf.append(u_seq[t])       # add feed into delay buffer
        feed = buf.pop(0)          # delayed feed arrives
        demand = demand_seq[t]
        tank = tank + feed - demand
        tank = max(0, min(tank, tank_volume))  # simulate overflow/underflow for realism
        cost += (tank_max - tank)**2 + 0.01*u_seq[t]**2  # penalize low tank + high feed
    return cost

# === MPC SIMULATION LOOP ===
for k in range(N_sim):
    # Demand forecast
    demand_seq = demand_profile[k:k+N_pred]
    if len(demand_seq) < N_pred:
        demand_seq = np.pad(demand_seq, (0, N_pred-len(demand_seq)), mode='edge')

    # Initial guess and bounds
    u0 = [300.0]*N_pred
    bounds = [(0, 600)] * N_pred  # max 600 m³/h feed capacity

    res = minimize(mpc_cost, u0, args=(tank_level, feed_delay_buffer.copy(), demand_seq),
                   bounds=bounds, method='SLSQP')

    u_opt = res.x
    u_now = u_opt[0] if res.success else 0.0
    feed_delay_buffer.append(u_now)
    feed = feed_delay_buffer.pop(0)

    demand = demand_profile[k]
    tank_level = tank_level + feed - demand
    tank_level = max(0, min(tank_level, tank_volume))

    # Log
    feed_log.append(u_now)
    tank_log.append(tank_level)

    # Console Output
    print(f"Hour {k:02d} | Feed: {u_now:.1f} m³/h | Demand: {demand:.1f} m³/h | Tank: {tank_level:.1f} m³")

# Optionally: Save logs to file or inspect tank_log, feed_log arrays
