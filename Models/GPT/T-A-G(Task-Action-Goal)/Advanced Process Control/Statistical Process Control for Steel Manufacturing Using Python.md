import numpy as np
import matplotlib.pyplot as plt

# Simulation parameters
samples = 25
subgroup_size = 5
np.random.seed(42)

# Simulate steel quality metrics
true_mean = 500  # MPa for tensile strength
true_std = 10     # standard deviation

# Subgroup data
data = np.random.normal(loc=true_mean, scale=true_std, size=(samples, subgroup_size))

# SPC chart calculations
xbar = np.mean(data, axis=1)
rbar = np.ptp(data, axis=1)

X_bar_bar = np.mean(xbar)
R_bar = np.mean(rbar)

# Constants for subgroup size 5
A2 = 0.577
D3 = 0
D4 = 2.114

# Control limits
UCL_x = X_bar_bar + A2 * R_bar
LCL_x = X_bar_bar - A2 * R_bar
UCL_r = D4 * R_bar
LCL_r = D3 * R_bar

# Check for violations
violations_x = np.where((xbar > UCL_x) | (xbar < LCL_x))[0]
violations_r = np.where((rbar > UCL_r) | (rbar < LCL_r))[0]

# Plot X-bar chart
plt.figure(figsize=(12, 5))
plt.subplot(1, 2, 1)
plt.plot(xbar, marker='o', label='X-bar')
plt.axhline(UCL_x, color='r', linestyle='--', label='UCL')
plt.axhline(LCL_x, color='r', linestyle='--', label='LCL')
plt.axhline(X_bar_bar, color='g', linestyle='-', label='Center')
plt.scatter(violations_x, xbar[violations_x], color='red', zorder=5, label='Violation')
plt.title('X̄ Chart - Tensile Strength')
plt.xlabel('Sample')
plt.ylabel('MPa')
plt.legend()
plt.grid(True)

# Plot R chart
plt.subplot(1, 2, 2)
plt.plot(rbar, marker='o', label='R')
plt.axhline(UCL_r, color='r', linestyle='--', label='UCL')
plt.axhline(LCL_r, color='r', linestyle='--', label='LCL')
plt.axhline(R_bar, color='g', linestyle='-', label='Center')
plt.scatter(violations_r, rbar[violations_r], color='red', zorder=5, label='Violation')
plt.title('R Chart - Tensile Strength')
plt.xlabel('Sample')
plt.ylabel('Range (MPa)')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.show()
