import numpy as np
import pandas as pd

np.random.seed(42)  # For reproducibility

def generate_process_data(n_samples=1000):
    # Simulating data for tensile strength (in MPa), thickness (in mm), and surface finish (Ra in μm)
    tensile_strength = np.random.normal(loc=500, scale=20, size=n_samples)  # Mean 500 MPa, Std Dev 20 MPa
    thickness = np.random.normal(loc=10, scale=0.5, size=n_samples)         # Mean 10 mm, Std Dev 0.5 mm
    surface_finish = np.random.normal(loc=1.5, scale=0.3, size=n_samples)   # Mean 1.5 μm, Std Dev 0.3 μm
    
    df = pd.DataFrame({
        'tensile_strength': tensile_strength,
        'thickness': thickness,
        'surface_finish': surface_finish
    })
    
    return df

data = generate_process_data()

from scipy.stats import norm

# Function to calculate control limits for X̄ and R charts
def calculate_control_limits(data, subgroup_size=5):
    means = data.groupby(np.arange(len(data)) // subgroup_size).mean()
    ranges = data.groupby(np.arange(len(data)) // subgroup_size).apply(lambda x: x.max() - x.min())
    
    x_bar = means.mean()
    r_bar = ranges.mean()
    
    A2 = 0.577  # Factor for calculating control limits, for subgroup size of 5
    D3, D4 = 0, 2.114  # Factors for R chart control limits
    
    x_upper_control_limit = x_bar + A2 * r_bar
    x_lower_control_limit = x_bar - A2 * r_bar
    
    r_upper_control_limit = D4 * r_bar
    r_lower_control_limit = D3 * r_bar
    
    return {
        'x_bar': x_bar, 'r_bar': r_bar,
        'x_upper_control_limit': x_upper_control_limit, 'x_lower_control_limit': x_lower_control_limit,
        'r_upper_control_limit': r_upper_control_limit, 'r_lower_control_limit': r_lower_control_limit
    }

control_limits = {col: calculate_control_limits(data[[col]]) for col in data.columns}

def check_control_limits(data, control_limits):
    alarms = []
    for feature in data.columns:
        cl = control_limits[feature]
        x_bar = cl['x_bar']
        
        # Check if any point is outside the control limits
        upper_alarm = data[feature] > cl['x_upper_control_limit']
        lower_alarm = data[feature] < cl['x_lower_control_limit']
        
        # Detect runs of 7 points above or below the mean
        runs_above = (data[feature] > x_bar).rolling(window=7).sum() == 7
        runs_below = (data[feature] < x_bar).rolling(window=7).sum() == 7
        
        alarms.append(pd.DataFrame({
            f'{feature}_upper_alarm': upper_alarm,
            f'{feature}_lower_alarm': lower_alarm,
            f'{feature}_runs_above': runs_above,
            f'{feature}_runs_below': runs_below
        }))
    
    alarms_df = pd.concat(alarms, axis=1)
    return alarms_df.any(axis=1)

alarms = check_control_limits(data, control_limits)
print("Alarms:", alarms.sum())  # Print total number of alarms detected

# Example: If alarms are detected, adjust the process to reduce variation
def adjust_process(data, feature, adjustment_factor=0.9):
    std_dev = data[feature].std()
    adjusted_std_dev = std_dev * adjustment_factor
    adjusted_feature = np.clip(np.random.normal(loc=data[feature].mean(), scale=adjusted_std_dev, size=len(data)), 
                               a_min=control_limits[feature]['x_lower_control_limit'], 
                               a_max=control_limits[feature]['x_upper_control_limit'])
    return adjusted_feature

# Apply adjustments for features where alarms were triggered
for feature in data.columns:
    if alarms[f'{feature}_upper_alarm'].any() or alarms[f'{feature}_lower_alarm'].any():
        data[feature] = adjust_process(data, feature)

# Recalculate control limits and check alarms again
new_control_limits = {col: calculate_control_limits(data[[col]]) for col in data.columns}
new_alarms = check_control_limits(data, new_control_limits)
print("New Alarms after Adjustment:", new_alarms.sum())
