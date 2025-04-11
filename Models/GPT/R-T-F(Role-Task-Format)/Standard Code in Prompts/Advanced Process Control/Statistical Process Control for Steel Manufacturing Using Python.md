```
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.stats.weightstats import zconfint

np.random.seed(42)

# Generate synthetic data
data_points = 100
tensile_strength = np.random.normal(loc=500, scale=50, size=data_points)  # Tensile strength in MPa
thickness = np.random.normal(loc=1.5, scale=0.1, size=data_points)  # Thickness in mm
surface_finish = np.random.normal(loc=0.5, scale=0.05, size=data_points)  # Surface finish in μm

# Introduce some out-of-control conditions
tensile_strength[20:30] += 100  # Increase tensile strength for samples 20 to 30
thickness[50:60] -= 0.3  # Decrease thickness for samples 50 to 60
surface_finish[70:80] += 0.2  # Increase surface finish for samples 70 to 80

# Combine into a DataFrame
df = pd.DataFrame({
    'Tensile_Strength': tensile_strength,
    'Thickness': thickness,
    'Surface_Finish': surface_finish
})
def create_control_chart(data, title, unit='units'):
    mean = data.mean()
    std_dev = data.std(ddof=0)  # Population standard deviation
    upper_control_limit = mean + 3 * std_dev
    lower_control_limit = mean - 3 * std_dev

    plt.figure(figsize=(10, 6))
    plt.plot(data.index, data.values, 'b-', label='Data')
    plt.axhline(y=mean, color='r', linestyle='--', label='Mean')
    plt.axhline(y=upper_control_limit, color='g', linestyle='--', label='UCL')
    plt.axhline(y=lower_control_limit, color='g', linestyle='--', label='LCL')
    plt.title(title)
    plt.xlabel('Sample Index')
    plt.ylabel(f'{unit}')
    plt.legend()
    plt.show()

    return mean, upper_control_limit, lower_control_limit

# Create control charts
create_control_chart(df['Tensile_Strength'], 'Control Chart for Tensile Strength', 'MPa')
create_control_chart(df['Thickness'], 'Control Chart for Thickness', 'mm')
create_control_chart(df['Surface_Finish'], 'Control Chart for Surface Finish', 'μm')
def check_out_of_control(data, mean, upper_control_limit, lower_control_limit):
    is_out_of_control = (data > upper_control_limit) | (data < lower_control_limit)
    return is_out_of_control

# Check for out-of-control conditions
tensile_strength_ucl, tensile_strength_lcl = create_control_chart(df['Tensile_Strength'], 'Control Chart for Tensile Strength', 'MPa')[:2]
thickness_ucl, thickness_lcl = create_control_chart(df['Thickness'], 'Control Chart for Thickness', 'mm')[:2]
surface_finish_ucl, surface_finish_lcl = create_control_chart(df['Surface_Finish'], 'Control Chart for Surface Finish', 'μm')[:2]

out_of_control_tensile = check_out_of_control(df['Tensile_Strength'], tensile_strength_ucl, tensile_strength_lcl)
out_of_control_thickness = check_out_of_control(df['Thickness'], thickness_ucl, thickness_lcl)
out_of_control_surface = check_out_of_control(df['Surface_Finish'], surface_finish_ucl, surface_finish_lcl)

# Alert when out-of-control conditions are detected
if out_of_control_tensile.any():
    print("ALERT: Tensile strength is out of control.")
if out_of_control_thickness.any():
    print("ALERT: Thickness is out of control.")
if out_of_control_surface.any():
    print("ALERT: Surface finish is out of control.")
def suggest_corrective_action(out_of_control, metric_name):
    if out_of_control.any():
        print(f"RECOMMENDATION FOR {metric_name}:")
        if out_of_control[20:30].all():  # Example assumption for tensile strength
            print("Adjust the alloy composition to improve tensile strength.")
        elif out_of_control[50:60].all():  # Example assumption for thickness
            print("Check the rolling mill settings to correct thickness issues.")
        elif out_of_control[70:80].all():  # Example assumption for surface finish
            print("Inspect the polishing equipment to reduce surface roughness.")

suggest_corrective_action(out_of_control_tensile, 'Tensile Strength')
suggest_corrective_action(out_of_control_thickness, 'Thickness')
suggest_corrective_action(out_of_control_surface, 'Surface Finish')
```
