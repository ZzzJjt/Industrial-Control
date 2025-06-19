import numpy as np
import matplotlib.pyplot as plt
import cvxpy as cp

# Simulation setup
dt = 1.0  # time step in minutes
T_sim = 240  # simulate for 4 hours
N = 20  # MPC horizon (20 min)
Q_target = 1000  # target buffer fill level (tons)
Q_max = 2000  # max tank capacity
feed_rate_max = 60  # max feed rate (tons/hr)
feed_rate_min = 20  # min feed rate (tons/hr)
upstream_rate = 50 / 60  # upstream rate in tons/min

# Time delay (2 hr = 120 min)
delay_steps = int(120 / dt)
delay_buffer = [upstream_rate] * delay_steps

# Initial states
Q = 1000  # initial buffer level
Q_list = [Q]
U_list = []
D_list = []

np.random.seed(0)

# Generate stochastic demand (tons/min)
demand_profile = 50 + 10 * np.sin(np.linspace(0, 2*np.pi, T_sim)) + np.random.randn(T_sim)
demand_profile = np.clip(demand_profile, 30, 70) / 60  # convert to tons/min

# MPC setup
for t in range(T_sim):
    # Update delay buffer
    delayed_in = delay_buffer.pop(0)

    # Define MPC variables
    u = cp.Variable(N)  # control: feed rate (tons/min)
    q = cp.Variable(N+1)  # predicted buffer level
    d_future = demand_profile[t:t+N] if t+N < T_sim else np.pad(demand_profile[t:], (0, N-(T_sim-t)), 'edge')

    # MPC objective and constraints
    constraints = [q[0] == Q]
    cost = 0
    for k in range(N):
        constraints += [q[k+1] == q[k] + delayed_in - d_future[k]]
        constraints += [u[k] >= feed_rate_min/60, u[k] <= feed_rate_max/60]
        cost += cp.square(q[k+1] - Q_target) + 0.1 * cp.square(u[k] - upstream_rate)

    prob = cp.Problem(cp.Minimize(cost), constraints)
    prob.solve()

    # Apply control
    u_apply = u.value[0]
    delay_buffer.append(u_apply)
    Q = Q + delayed_in - demand_profile[t]
    Q = np.clip(Q, 0, Q_max)

    Q_list.append(Q)
    U_list.append(u_apply)
    D_list.append(demand_profile[t])

# Plot results
time = np.arange(0, T_sim+1) * dt
plt.figure(figsize=(12, 6))

plt.subplot(2, 1, 1)
plt.plot(time, Q_list, label='Buffer Level (tons)')
plt.axhline(Q_target, color='gray', linestyle='--', label='Target Level')
plt.axhline(Q_max, color='red', linestyle='--', label='Tank Capacity')
plt.grid(); plt.legend(); plt.ylabel('Buffer Level (tons)'); plt.title('Buffer Tank Dynamics')

plt.subplot(2, 1, 2)
plt.step(time[:-1], np.array(U_list)*60, label='Feed Rate (tons/hr)')
plt.step(time[:-1], np.array(D_list)*60, label='Demand (tons/hr)', linestyle='--')
plt.grid(); plt.legend(); plt.xlabel('Time (min)'); plt.ylabel('Flow Rate (tons/hr)')

plt.tight_layout()
plt.show()
