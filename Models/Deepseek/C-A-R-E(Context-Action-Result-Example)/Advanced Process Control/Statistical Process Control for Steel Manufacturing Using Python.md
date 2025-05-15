import numpy as np
import matplotlib.pyplot as plt

# Constants
num_samples = 100  # Total number of samples
subgroup_size = 5  # Size of each subgroup
num_subgroups = num_samples // subgroup_size  # Number of subgroups

# Simulate thickness measurements with gradual drift
np.random.seed(42)
thickness_mean = 5.0  # Initial mean thickness
thickness_std = 0.1   # Standard deviation of thickness
drift_start = 30      # Time step when drift starts
drift_rate = 0.01     # Rate of drift

thickness_data = []
for t in range(num_samples):
    if t >= drift_start:
        thickness_mean += drift_rate
    thickness_sample = np.random.normal(thickness_mean, thickness_std, subgroup_size)
    thickness_data.append(thickness_sample)

thickness_data = np.array(thickness_data)

# Calculate means and ranges for each subgroup
thickness_means = np.mean(thickness_data, axis=1)
thickness_ranges = np.ptp(thickness_data, axis=1)
thickness_stds = np.std(thickness_data, axis=1, ddof=1)

# Calculate overall mean and control limits for X̄ chart
overall_mean = np.mean(thickness_means)
R_bar = np.mean(thickness_ranges)
S_bar = np.mean(thickness_stds)

A2 = 0.577  # Control limit factor for X̄ chart with subgroup size 5
B3 = 0.228  # Control limit factor for R chart with subgroup size 5
B4 = 2.282  # Control limit factor for R chart with subgroup size 5
B5 = 0.308  # Control limit factor for S chart with subgroup size 5
B6 = 1.128  # Control limit factor for S chart with subgroup size 5

Xbar_UCL = overall_mean + A2 * R_bar
Xbar_LCL = overall_mean - A2 * R_bar
R_UCL = B4 * R_bar
R_LCL = B3 * R_bar
S_UCL = B6 * S_bar
S_LCL = B5 * S_bar

# Plot X̄ chart
plt.figure(figsize=(14, 10))
plt.subplot(3, 1, 1)
plt.plot(range(1, num_subgroups + 1), thickness_means, marker='o', label='Subgroup Mean')
plt.axhline(y=overall_mean, color='r', linestyle='--', label='Overall Mean')
plt.axhline(y=Xbar_UCL, color='g', linestyle='-.', label='UCL')
plt.axhline(y=Xbar_LCL, color='g', linestyle='-.', label='LCL')
plt.xlabel('Subgroup')
plt.ylabel('Mean Thickness')
plt.title('X̄ Chart for Thickness')
plt.legend()

# Plot R chart
plt.subplot(3, 1, 2)
plt.plot(range(1, num_subgroups + 1), thickness_ranges, marker='o', label='Subgroup Range')
plt.axhline(y=R_bar, color='r', linestyle='--', label='Average Range')
plt.axhline(y=R_UCL, color='g', linestyle='-.', label='UCL')
plt.axhline(y=R_LCL, color='g', linestyle='-.', label='LCL')
plt.xlabel('Subgroup')
plt.ylabel('Range of Thickness')
plt.title('R Chart for Thickness')
plt.legend()

# Plot S chart
plt.subplot(3, 1, 3)
plt.plot(range(1, num_subgroups + 1), thickness_stds, marker='o', label='Subgroup Std Deviation')
plt.axhline(y=S_bar, color='r', linestyle='--', label='Average Std Deviation')
plt.axhline(y=S_UCL, color='g', linestyle='-.', label='UCL')
plt.axhline(y=S_LCL, color='g', linestyle='-.', label='LCL')
plt.xlabel('Subgroup')
plt.ylabel('Standard Deviation of Thickness')
plt.title('S Chart for Thickness')
plt.legend()

plt.tight_layout()
plt.show()

# Detect out-of-control conditions and suggest corrective actions
out_of_control = False
corrective_actions = []

for i in range(num_subgroups):
    if thickness_means[i] > Xbar_UCL or thickness_means[i] < Xbar_LCL:
        out_of_control = True
        corrective_actions.append(f"Subgroup {i+1}: Out-of-control condition detected in mean thickness. Adjust rolling pressure.")
    if thickness_ranges[i] > R_UCL or thickness_ranges[i] < R_LCL:
        out_of_control = True
        corrective_actions.append(f"Subgroup {i+1}: Out-of-control condition detected in range of thickness. Check machinery alignment.")
    if thickness_stds[i] > S_UCL or thickness_stds[i] < S_LCL:
        out_of_control = True
        corrective_actions.append(f"Subgroup {i+1}: Out-of-control condition detected in standard deviation of thickness. Verify raw material quality.")

if out_of_control:
    print("Out-of-control conditions detected:")
    for action in corrective_actions:
        print(action)
else:
    print("No out-of-control conditions detected.")



