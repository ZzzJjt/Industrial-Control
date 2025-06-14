import numpy as np
import matplotlib.pyplot as plt
from scipy.linalg import solve_discrete_are

# Simulation parameters
sim_time = 100  # in hours
delay = 2       # batch delay in hours
buffer_capacity = 1000  # max buffer level

# System dynamics: Buffer level depends on infeed - outfeed
# x[k+1] = x[k] + infeed[k-delay] - outfeed[k]
# We'll simulate this with shifted arrays

# Generate random demand (outfeed)
outfeed = 50 + 20 * np.sin(np.linspace(0, 10, sim_time)) + np.random.randn(sim_time) * 5
outfeed = np.clip(outfeed, 30, 70)

# Desired buffer level
target_buffer = 800

# Initialize variables
buffer = np.zeros(sim_time)
infeed = np.zeros(sim_time)
delayed_infeed = np.zeros(sim_time)

# Define MPC parameters
Q = 1.0  # Weight on buffer level error
R = 0.1  # Weight on infeed change
A = 1
B = 1
N = 10  # Prediction horizon

# Solve discrete-time Riccati equation for LQR gain as simple MPC approximation
P = solve_discrete_are(A, B, Q, R)
K = -np.linalg.inv(R + B.T @ P @ B) @ B.T @ P @ A

# Main simulation loop
for k in range(sim_time - 1):
    # Compute error from target
    error = buffer[k] - target_buffer

    # MPC-like control law
    u = float(K * error)
    infeed[k] = np.clip(u + outfeed[k], 0, 100)  # anticipate demand

    # Implement delay logic
    if k >= delay:
        delayed_infeed[k] = infeed[k - delay]
    else:
        delayed_infeed[k] = 0

    # Update buffer level
    buffer[k + 1] = buffer[k] + delayed_infeed[k] - outfeed[k]
    buffer[k + 1] = np.clip(buffer[k + 1], 0, buffer_capacity)

# Plot results
plt.figure(figsize=(12, 6))
plt.plot(buffer, label='Buffer Level')
plt.axhline(target_buffer, color='gray', linestyle='--', label='Target Level')
plt.plot(infeed, label='Infeed Rate')
plt.plot(outfeed, label='Outfeed Rate')
plt.title('MPC Control of Cellulose Buffer Tank')
plt.xlabel('Time (hours)')
plt.ylabel('Units')
plt.legend()
plt.grid(True)
plt.show()
