import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import norm

# Constants
num_samples = 50  # Total number of samples
sample_size = 5   # Sample size for each subgroup
num_subgroups = num_samples // sample_size  # Number of subgroups

# Parameters for normal distribution
mean_strength = 500  # Mean tensile strength [MPa]
std_strength = 10    # Standard deviation tensile strength [MPa]

mean_thickness = 10  # Mean thickness [mm]
std_thickness = 0.1  # Standard deviation thickness [mm]

mean_finish = 85     # Mean surface finish [Ra micrometers]
std_finish = 2       # Standard deviation surface finish [Ra micrometers]

# Simulate data
np.random.seed(42)
strength_data = np.random.normal(mean_strength, std_strength, num_samples)
thickness_data = np.random.normal(mean_thickness, std_thickness, num_samples)
finish_data = np.random.normal(mean_finish, std_finish, num_samples)

# Organize data into subgroups
strength_subgroups = strength_data.reshape(num_subgroups, sample_size)
thickness_subgroups = thickness_data.reshape(num_subgroups, sample_size)
finish_subgroups = finish_data.reshape(num_subgroups, sample_size)

# Calculate means and ranges for each subgroup
strength_means = np.mean(strength_subgroups, axis=1)
strength_ranges = np.ptp(strength_subgroups, axis=1)

thickness_means = np.mean(thickness_subgroups, axis=1)
thickness_ranges = np.ptp(thickness_subgroups, axis=1)

finish_means = np.mean(finish_subgroups, axis=1)
finish_ranges = np.ptp(finish_subgroups, axis=1)

# Calculate overall mean and standard deviation
overall_mean_strength = np.mean(strength_means)
overall_std_strength = np.std(strength_means, ddof=1)

overall_mean_thickness = np.mean(thickness_means)
overall_std_thickness = np.std(thickness_means, ddof=1)

overall_mean_finish = np.mean(finish_means)
overall_std_finish = np.std(finish_means, ddof=1)

# Calculate control limits for X̄ chart
A2 = 0.577  # Constant for n=5
D3 = 0.000  # Constant for n=5
D4 = 2.114  # Constant for n=5

strength_xbar_cl = overall_mean_strength
strength_xbar_ucl = overall_mean_strength + A2 * overall_std_strength
strength_xbar_lcl = overall_mean_strength - A2 * overall_std_strength

thickness_xbar_cl = overall_mean_thickness
thickness_xbar_ucl = overall_mean_thickness + A2 * overall_std_thickness
thickness_xbar_lcl = overall_mean_thickness - A2 * overall_std_thickness

finish_xbar_cl = overall_mean_finish
finish_xbar_ucl = overall_mean_finish + A2 * overall_std_finish
finish_xbar_lcl = overall_mean_finish - A2 * overall_std_finish

# Calculate control limits for R chart
strength_r_cl = np.mean(strength_ranges)
strength_r_ucl = D4 * strength_r_cl
strength_r_lcl = D3 * strength_r_cl

thickness_r_cl = np.mean(thickness_ranges)
thickness_r_ucl = D4 * thickness_r_cl
thickness_r_lcl = D3 * thickness_r_cl

finish_r_cl = np.mean(finish_ranges)
finish_r_ucl = D4 * finish_r_cl
finish_r_lcl = D3 * finish_r_cl

# Western Electric Rules for detecting out-of-control signals
def detect_out_of_control(subgroup_means, cl, ucl, lcl):
    alarms = []
    for i in range(len(subgroup_means)):
        if subgroup_means[i] > ucl or subgroup_means[i] < lcl:
            alarms.append(i)
    return alarms

# Function to suggest corrective actions
def suggest_corrective_actions(metric_name, alarms):
    if not alarms:
        print(f"No out-of-control conditions detected for {metric_name}.")
    else:
        print(f"Out-of-control conditions detected for {metric_name} at subgroups: {alarms}")
        for alarm in alarms:
            print(f"Subgroup {alarm}: Investigate potential causes such as machine malfunctions or material issues.")
            print(f"Suggested corrective actions:")
            print("1. Inspect and calibrate machinery.")
            print("2. Review raw material quality.")
            print("3. Conduct process audits.")
            print("----------------------------------------")

# Detect out-of-control signals for X̄ charts
strength_alarms = detect_out_of_control(strength_means, strength_xbar_cl, strength_xbar_ucl, strength_xbar_lcl)
thickness_alarms = detect_out_of_control(thickness_means, thickness_xbar_cl, thickness_xbar_ucl, thickness_xbar_lcl)
finish_alarms = detect_out_of_control(finish_means, finish_xbar_cl, finish_xbar_ucl, finish_xbar_lcl)

# Suggest corrective actions
suggest_corrective_actions("Tensile Strength", strength_alarms)
suggest_corrective_actions("Thickness", thickness_alarms)
suggest_corrective_actions("Surface Finish", finish_alarms)

# Optional: Plotting
plt.figure(figsize=(16, 10))

# Plot Tensile Strength X̄ Chart
plt.subplot(3, 2, 1)
plt.plot(range(num_subgroups), strength_means, marker='o', label='X̄')
plt.axhline(y=strength_xbar_cl, color='r', linestyle='--', label='CL')
plt.axhline(y=strength_xbar_ucl, color='g', linestyle='--', label='UCL')
plt.axhline(y=strength_xbar_lcl, color='b', linestyle='--', label='LCL')
for alarm in strength_alarms:
    plt.scatter(alarm, strength_means[alarm], color='red', zorder=5)
plt.xlabel('Subgroup')
plt.ylabel('Mean Tensile Strength (MPa)')
plt.title('X̄ Chart for Tensile Strength')
plt.legend()

# Plot Tensile Strength R Chart
plt.subplot(3, 2, 2)
plt.plot(range(num_subgroups), strength_ranges, marker='o', label='R')
plt.axhline(y=strength_r_cl, color='r', linestyle='--', label='CL')
plt.axhline(y=strength_r_ucl, color='g', linestyle='--', label='UCL')
plt.axhline(y=strength_r_lcl, color='b', linestyle='--', label='LCL')
for alarm in strength_alarms:
    plt.scatter(alarm, strength_ranges[alarm], color='red', zorder=5)
plt.xlabel('Subgroup')
plt.ylabel('Range Tensile Strength (MPa)')
plt.title('R Chart for Tensile Strength')
plt.legend()

# Plot Thickness X̄ Chart
plt.subplot(3, 2, 3)
plt.plot(range(num_subgroups), thickness_means, marker='o', label='X̄')
plt.axhline(y=thickness_xbar_cl, color='r', linestyle='--', label='CL')
plt.axhline(y=thickness_xbar_ucl, color='g', linestyle='--', label='UCL')
plt.axhline(y=thickness_xbar_lcl, color='b', linestyle='--', label='LCL')
for alarm in thickness_alarms:
    plt.scatter(alarm, thickness_means[alarm], color='red', zorder=5)
plt.xlabel('Subgroup')
plt.ylabel('Mean Thickness (mm)')
plt.title('X̄ Chart for Thickness')
plt.legend()

# Plot Thickness R Chart
plt.subplot(3, 2, 4)
plt.plot(range(num_subgroups), thickness_ranges, marker='o', label='R')
plt.axhline(y=thickness_r_cl, color='r', linestyle='--', label='CL')
plt.axhline(y=thickness_r_ucl, color='g', linestyle='--', label='UCL')
plt.axhline(y=thickness_r_lcl, color='b', linestyle='--', label='LCL')
for alarm in thickness_alarms:
    plt.scatter(alarm, thickness_ranges[alarm], color='red', zorder=5)
plt.xlabel('Subgroup')
plt.ylabel('Range Thickness (mm)')
plt.title('R Chart for Thickness')
plt.legend()

# Plot Surface Finish X̄ Chart
plt.subplot(3, 2, 5)
plt.plot(range(num_subgroups), finish_means, marker='o', label='X̄')
plt.axhline(y=finish_xbar_cl, color='r', linestyle='--', label='CL')
plt.axhline(y=finish_xbar_ucl, color='g', linestyle='--', label='UCL')
plt.axhline(y=finish_xbar_lcl, color='b', linestyle='--', label='LCL')
for alarm in finish_alarms:
    plt.scatter(alarm, finish_means[alarm], color='red', zorder=5)
plt.xlabel('Subgroup')
plt.ylabel('Mean Surface Finish (Ra μm)')
plt.title('X̄ Chart for Surface Finish')
plt.legend()

# Plot Surface Finish R Chart
plt.subplot(3, 2, 6)
plt.plot(range(num_subgroups), finish_ranges, marker='o', label='R')
plt.axhline(y=finish_r_cl, color='r', linestyle='--', label='CL')
plt.axhline(y=finish_r_ucl, color='g', linestyle='--', label='UCL')
plt.axhline(y=finish_r_lcl, color='b', linestyle='--', label='LCL')
for alarm in finish_alarms:
    plt.scatter(alarm, finish_ranges[alarm], color='red', zorder=5)
plt.xlabel('Subgroup')
plt.ylabel('Range Surface Finish (Ra μm)')
plt.title('R Chart for Surface Finish')
plt.legend()

plt.tight_layout()
plt.show()
