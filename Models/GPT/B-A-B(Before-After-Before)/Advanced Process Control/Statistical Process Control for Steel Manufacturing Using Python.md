import numpy as np
import matplotlib.pyplot as plt

# Configuration
num_batches = 50
samples_per_batch = 5
target_strength = 450  # MPa
std_dev = 10

# Control chart constants for X̄ and R charts (n = 5)
A2 = 0.577
D3 = 0
D4 = 2.114

# Simulate batches with slight process drift and occasional disturbance
np.random.seed(0)
batch_data = []
for i in range(num_batches):
    drift = 0.1 * i  # small upward drift
    disturbance = 20 * np.random.randn() if i in [20, 35] else 0
    batch = np.random.normal(target_strength + drift + disturbance, std_dev, samples_per_batch)
    batch_data.append(batch)

# Calculate X̄ and R values for each batch
xbar = np.array([np.mean(batch) for batch in batch_data])
rbar = np.array([np.max(batch) - np.min(batch) for batch in batch_data])

# Compute control limits
xbar_bar = np.mean(xbar)
rbar_bar = np.mean(rbar)

ucl_xbar = xbar_bar + A2 * rbar_bar
lcl_xbar = xbar_bar - A2 * rbar_bar
ucl_r = D4 * rbar_bar
lcl_r = D3 * rbar_bar

# Detect out-of-control points
alerts = []
for i in range(num_batches):
    if xbar[i] > ucl_xbar or xbar[i] < lcl_xbar:
        alerts.append((i, 'X̄ chart breach'))
    elif rbar[i] > ucl_r or rbar[i] < lcl_r:
        alerts.append((i, 'R chart breach'))

# Plot X̄ chart
plt.figure(figsize=(10, 6))
plt.subplot(2,1,1)
plt.plot(xbar, marker='o', label='X̄')
plt.axhline(ucl_xbar, color='r', linestyle='--', label='UCL')
plt.axhline(lcl_xbar, color='r', linestyle='--', label='LCL')
plt.axhline(xbar_bar, color='g', linestyle='--', label='Center')
plt.title('X̄ Chart - Tensile Strength (MPa)')
plt.ylabel('Mean Strength')
plt.grid(True)
plt.legend()

# Plot R chart
plt.subplot(2,1,2)
plt.plot(rbar, marker='o', label='R')
plt.axhline(ucl_r, color='r', linestyle='--', label='UCL')
plt.axhline(lcl_r, color='r', linestyle='--', label='LCL')
plt.axhline(rbar_bar, color='g', linestyle='--', label='Center')
plt.title('R Chart - Range of Strength')
plt.xlabel('Batch Number')
plt.ylabel('Range')
plt.grid(True)
plt.legend()

plt.tight_layout()
plt.show()

# Output alerts
print("\n📢 Alerts:")
for idx, reason in alerts:
    print(f"Batch {idx + 1}: {reason} — suggest checking rolling pressure, temperature control, or raw material consistency.")
