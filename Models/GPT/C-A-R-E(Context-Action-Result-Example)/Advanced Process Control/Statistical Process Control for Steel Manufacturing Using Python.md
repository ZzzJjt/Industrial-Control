import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import norm

# Simulated data generation
np.random.seed(0)
samples = 100
subgroup_size = 5

# Mean and standard deviation for quality metrics
mean_thickness = 6.0     # mm
std_thickness = 0.1      # mm
mean_tensile = 450       # MPa
std_tensile = 15         # MPa
mean_surface = 80        # %
std_surface = 5          # %

# Generate subgroups of quality metric data
def generate_data(mean, std):
    return np.random.normal(loc=mean, scale=std, size=(samples, subgroup_size))

thickness_data = generate_data(mean_thickness, std_thickness)
tensile_data = generate_data(mean_tensile, std_tensile)
surface_data = generate_data(mean_surface, std_surface)

# Control chart function (X̄ and R chart)
def spc_control_chart(data, metric_name):
    xbar = np.mean(data, axis=1)
    R = np.ptp(data, axis=1)

    xbar_bar = np.mean(xbar)
    R_bar = np.mean(R)

    A2 = 0.577  # for subgroup size of 5
    UCL = xbar_bar + A2 * R_bar
    LCL = xbar_bar - A2 * R_bar

    # Detect violations
    out_of_control = np.where((xbar > UCL) | (xbar < LCL))[0]

    # Plot
    plt.figure(figsize=(10, 4))
    plt.plot(xbar, marker='o', label=f'{metric_name} mean')
    plt.axhline(xbar_bar, color='green', linestyle='--', label='CL')
    plt.axhline(UCL, color='red', linestyle='--', label='UCL')
    plt.axhline(LCL, color='red', linestyle='--', label='LCL')
    plt.scatter(out_of_control, xbar[out_of_control], color='red', zorder=5, label='Out of control')
    plt.title(f'X̄ Control Chart for {metric_name}')
    plt.xlabel('Subgroup Index')
    plt.ylabel(metric_name)
    plt.legend()
    plt.grid(True)
    plt.tight_layout()
    plt.show()

    # Print recommendations
    if len(out_of_control) > 0:
        print(f"[ALERT] {metric_name} out-of-control points at:", out_of_control)
        print(f"[SUGGESTION] Investigate related process parameter and adjust accordingly.")
    else:
        print(f"[STATUS] {metric_name} is within control limits.")

# Run control charts for each metric
spc_control_chart(thickness_data, "Thickness (mm)")
spc_control_chart(tensile_data, "Tensile Strength (MPa)")
spc_control_chart(surface_data, "Surface Finish (%)")
