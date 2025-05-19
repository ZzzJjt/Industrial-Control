import numpy as np
import pandas as pd

# Parameters
n_samples = 1000  # Total subgroups
subgroup_size = 5  # Samples per subgroup
sample_interval = 60  # Seconds between subgroups
np.random.seed(42)

# Target and tolerance for metrics
metrics = {
    'tensile_strength': {'target': 500.0, 'tol': 50.0, 'std': 10.0, 'unit': 'MPa'},
    'thickness': {'target': 2.0, 'tol': 0.2, 'std': 0.05, 'unit': 'mm'},
    'surface_finish': {'target': 1.0, 'tol': 0.3, 'std': 0.05, 'unit': 'Ra µm'}
}

# 1. Simulate Steel Manufacturing Data
def generate_process_data(n_samples, subgroup_size, metrics):
    data = {key: [] for key in metrics}
    time = np.arange(n_samples) * sample_interval / 3600  # Time in hours
    
    for metric, params in metrics.items():
        # Base process: Normal distribution
        values = np.random.normal(params['target'], params['std'], (n_samples, subgroup_size))
        
        # Introduce drift and shift
        if metric == 'thickness':
            drift = 0.1 * time  # +0.1 mm/hour
            values += drift[:, np.newaxis]
        elif metric == 'tensile_strength':
            shift = np.where((time > 5) & (time < 7), 30.0, 0.0)  # +30 MPa at 5–7 hours
            values += shift[:, np.newaxis]
        elif metric == 'surface_finish':
            shift = np.where((time > 10) & (time < 12), 0.2, 0.0)  # +0.2 Ra µm at 10–12 hours
            values += shift[:, np.newaxis]
        
        data[metric] = values
    
    return data, time

data, time = generate_process_data(n_samples, subgroup_size, metrics)

# 2. SPC Tools: X̄ and R Charts
def calculate_control_limits(data, subgroup_size):
    limits = {}
    for metric, values in data.items():
        # Subgroup statistics
        X_bar = np.mean(values, axis=1)  # Subgroup means
        R = np.ptp(values, axis=1)       # Subgroup ranges
        
        # Grand averages
        X_double_bar = np.mean(X_bar)
        R_bar = np.mean(R)
        
        # Control limits
        n = subgroup_size
        A2 = 0.577  # For n=5
        D3 = 0.0    # For n=5
        D4 = 2.114  # For n=5
        sigma_X = R_bar / 2.326  # Approximate sigma for n=5
        
        limits[metric] = {
            'X_bar': X_bar,
            'R': R,
            'X_CL': X_double_bar,
            'X_UCL': X_double_bar + A2 * R_bar,
            'X_LCL': X_double_bar - A2 * R_bar,
            'R_CL': R_bar,
            'R_UCL': D4 * R_bar,
            'R_LCL': D3 * R_bar,
            'sigma': sigma_X
        }
    
    return limits

limits = calculate_control_limits(data, subgroup_size)

# Western Electric Rules
def detect_out_of_control(X_bar, R, limits, window=7):
    violations = []
    for i in range(len(X_bar)):
        # Rule 1: Point beyond UCL/LCL
        if X_bar[i] > limits['X_UCL'] or X_bar[i] < limits['X_LCL']:
            violations.append((i, 'Rule 1: Beyond UCL/LCL'))
        elif R[i] > limits['R_UCL'] or R[i] < limits['R_LCL']:
            violations.append((i, 'Rule 1: Range beyond UCL/LCL'))
        
        # Rule 2: 7 consecutive points above/below centerline
        if i >= window - 1:
            if all(X_bar[i-window+1:i+1] > limits['X_CL']):
                violations.append((i, 'Rule 2: 7 above centerline'))
            elif all(X_bar[i-window+1:i+1] < limits['X_CL']):
                violations.append((i, 'Rule 2: 7 below centerline'))
        
        # Rule 3: 7 consecutive points increasing/decreasing
        if i >= window - 1:
            diffs = np.diff(X_bar[i-window+1:i+1])
            if all(diffs > 0):
                violations.append((i, 'Rule 3: 7 increasing'))
            elif all(diffs < 0):
                violations.append((i, 'Rule 3: 7 decreasing'))
    
    return violations

# 3. Alarm and Corrective Actions
def trigger_alarms_and_actions(data, limits, time):
    alarms = {metric: [] for metric in data}
    corrective_actions = []
    
    for metric, values in data.items():
        X_bar = limits[metric]['X_bar']
        R = limits[metric]['R']
        violations = detect_out_of_control(X_bar, R, limits[metric])
        
        for idx, reason in violations:
            t = time[idx]
            mean = X_bar[idx]
            range_val = R[idx]
            action = suggest_corrective_action(metric, reason, mean, limits[metric])
            alarms[metric].append({
                'time': t,
                'reason': reason,
                'mean': mean,
                'range': range_val,
                'action': action
            })
            corrective_actions.append((idx, metric, action))
    
    return alarms, corrective_actions

def suggest_corrective_action(metric, reason, mean, limits):
    if 'tensile_strength' in metric:
        if 'above' in reason or mean > limits['X_CL']:
            return "Reduce annealing temperature by 5°C or adjust quenching rate."
        elif 'below' in reason or mean < limits['X_CL']:
            return "Increase annealing temperature by 5°C or check alloy composition."
    elif 'thickness' in metric:
        if 'above' in reason or mean > limits['X_CL']:
            return "Increase rolling pressure by 10% or adjust roller gap."
        elif 'below' in reason or mean < limits['X_CL']:
            return "Decrease rolling pressure by 10% or check feed rate."
    elif 'surface_finish' in metric:
        if 'above' in reason or mean > limits['X_CL']:
            return "Polish rolls or reduce rolling speed by 5%."
        elif 'below' in reason or mean < limits['X_CL']:
            return "Check lubricant application or roll condition."
    return "Inspect process parameters."

alarms, corrective_actions = trigger_alarms_and_actions(data, limits, time)

# 4. Evaluate Performance with Interventions
def simulate_with_interventions(data, corrective_actions, metrics):
    data_corrected = {key: data[key].copy() for key in data}
    
    for idx, metric, action in corrective_actions:
        # Apply correction: Shift mean back toward target
        if idx < n_samples - 1:
            shift = limits[metric]['X_CL'] - limits[metric]['X_bar'][idx]
            data_corrected[metric][idx+1:] += shift
    
    return data_corrected

data_corrected = simulate_with_interventions(data, corrective_actions, metrics)

# Calculate Defect Rates and Variation
def calculate_metrics(data, metrics):
    defects = {key: 0 for key in metrics}
    variation = {key: 0.0 for key in metrics}
    
    for metric, values in data.items():
        target = metrics[metric]['target']
        tol = metrics[metric]['tol']
        for subgroup in values:
            for value in subgroup:
                if value < target - tol or value > target + tol:
                    defects[metric] += 1
        variation[metric] = np.std(values)
    
    return defects, variation

defects_orig, variation_orig = calculate_metrics(data, metrics)
defects_corr, variation_corr = calculate_metrics(data_corrected, metrics)

# Output Results
print("SPC System Results:")
for metric in metrics:
    print(f"\nMetric: {metric} ({metrics[metric]['unit']})")
    print(f"Original Defects: {defects_orig[metric]} (Rate: {defects_orig[metric]/(n_samples*subgroup_size)*100:.2f}%)")
    print(f"Corrected Defects: {defects_corr[metric]} (Rate: {defects_corr[metric]/(n_samples*subgroup_size)*100:.2f}%)")
    print(f"Original Variation (Std): {variation_orig[metric]:.3f} {metrics[metric]['unit']}")
    print(f"Corrected Variation (Std): {variation_corr[metric]:.3f} {metrics[metric]['unit']}")
    print("Sample Alarms:")
    for alarm in alarms[metric][:3]:  # Show up to 3 alarms
        print(f"Time: {alarm['time']:.2f}h, Reason: {alarm['reason']}, "
              f"Mean: {alarm['mean']:.2f}, Action: {alarm['action']}")

# Calculate Waste Reduction
total_defects_orig = sum(defects_orig.values())
total_defects_corr = sum(defects_corr.values())
waste_reduction = (total_defects_orig - total_defects_corr) / total_defects_orig * 100
print(f"\nOverall Waste Reduction: {waste_reduction:.2f}%")
