import numpy as np
import pandas as pd
from datetime import datetime

# Step 1: Define System Parameters
SAMPLE_SIZE = 5  # Number of samples per subgroup
SUBGROUP_FREQ = 60  # Seconds between subgroups
SIM_DURATION = 3600 * 2  # 2 hours simulation (seconds)
N_SUBGROUPS = int(SIM_DURATION / SUBGROUP_FREQ)  # Number of subgroups
METRICS = ['Tensile_Strength_MPa', 'Thickness_mm', 'Surface_Finish_Ra']
NOMINAL = {
    'Tensile_Strength_MPa': 500.0,  # Nominal value
    'Thickness_mm': 2.0,
    'Surface_Finish_Ra': 1.6  # Micrometers
}
STD_DEV = {
    'Tensile_Strength_MPa': 10.0,  # Standard deviation under control
    'Thickness_mm': 0.05,
    'Surface_Finish_Ra': 0.1
}

# Step 2: Simulate Steel Manufacturing Data
def simulate_data(n_subgroups, sample_size, metrics, nominal, std_dev):
    """Simulate quality metrics with normal and out-of-control conditions"""
    data = {metric: [] for metric in metrics}
    timestamps = []
    np.random.seed(42)
    
    for i in range(n_subgroups):
        t = i * SUBGROUP_FREQ
        timestamps.append(datetime.fromtimestamp(t).strftime('%Y-%m-%d %H:%M:%S'))
        
        # Introduce out-of-control conditions
        shift = 0.0
        drift = 0.0
        if 30 <= i < 40:  # Temporary shift (e.g., tool wear)
            shift = 2.0 * std_dev[metrics[0]]
        if i >= 60:  # Gradual drift (e.g., calibration issue)
            drift = 0.05 * std_dev[metrics[0]] * (i - 60)
        
        for metric in metrics:
            # Normal process with noise
            samples = np.random.normal(
                nominal[metric] + shift + drift,
                std_dev[metric],
                sample_size
            )
            data[metric].append(samples)
    
    return pd.DataFrame({
        'Timestamp': timestamps,
        **{metric: data[metric] for metric in metrics}
    })

# Step 3: SPC Chart Calculations
def calculate_control_limits(data, metric, sample_size):
    """Calculate X̄ and R chart control limits"""
    subgroups = data[metric].values
    x_bar = np.mean(subgroups, axis=1)  # Subgroup means
    ranges = np.ptp(subgroups, axis=1)  # Subgroup ranges
    
    # Constants for control limits (A2, D3, D4 from SPC tables for n=5)
    A2 = 0.577
    D3 = 0.0
    D4 = 2.114
    
    # X̄ chart limits
    x_bar_mean = np.mean(x_bar)
    R_mean = np.mean(ranges)
    UCL_x = x_bar_mean + A2 * R_mean
    LCL_x = x_bar_mean - A2 * R_mean
    
    # R chart limits
    UCL_r = D4 * R_mean
    LCL_r = D3 * R_mean
    
    return {
        'x_bar': x_bar,
        'ranges': ranges,
        'x_bar_mean': x_bar_mean,
        'R_mean': R_mean,
        'UCL_x': UCL_x,
        'LCL_x': LCL_x,
        'UCL_r': UCL_r,
        'LCL_r': LCL_r
    }

# Step 4: Out-of-Control Detection (Western Electric Rules)
def detect_violations(x_bar, ranges, limits, metric):
    """Detect out-of-control conditions using Western Electric rules"""
    violations = []
    x_bar_mean = limits['x_bar_mean']
    R_mean = limits['R_mean']
    sigma_x = (limits['UCL_x'] - x_bar_mean) / 3  # Approximate sigma for X̄ chart
    
    for i in range(len(x_bar)):
        violation = None
        # Rule 1: Point beyond 3-sigma limits
        if x_bar[i] > limits['UCL_x'] or x_bar[i] < limits['LCL_x']:
            violation = f"{metric} X̄ beyond 3-sigma limits"
        elif ranges[i] > limits['UCL_r'] or ranges[i] < limits['LCL_r']:
            violation = f"{metric} R beyond control limits"
        # Rule 2: 2 out of 3 points beyond 2-sigma
        if i >= 2:
            beyond_2s = sum(1 for j in range(i-2, i+1) if
                           abs(x_bar[j] - x_bar_mean) > 2 * sigma_x)
            if beyond_2s >= 2:
                violation = f"{metric} X̄ 2/3 points beyond 2-sigma"
        # Rule 3: 4 out of 5 points beyond 1-sigma
        if i >= 4:
            beyond_1s = sum(1 for j in range(i-4, i+1) if
                           abs(x_bar[j] - x_bar_mean) > sigma_x)
            if beyond_1s >= 4:
                violation = f"{metric} X̄ 4/5 points beyond 1-sigma"
        # Rule 4: 8 consecutive points on one side of centerline
        if i >= 7:
            same_side = all(x_bar[j] > x_bar_mean for j in range(i-7, i+1)) or \
                        all(x_bar[j] < x_bar_mean for j in range(i-7, i+1))
            if same_side:
                violation = f"{metric} X̄ 8 points on one side"
        
        if violation:
            violations.append((i, violation))
    
    return violations

# Step 5: Alarm and Corrective Actions
def trigger_alarm(violations, data, metric, limits, corrective_actions):
    """Trigger alarms and suggest corrective actions"""
    alarms = []
    for idx, violation in violations:
        timestamp = data['Timestamp'][idx]
        x_bar = limits['x_bar'][idx]
        range_val = limits['ranges'][idx]
        action = corrective_actions.get(violation, "Investigate process")
        alarm = {
            'Timestamp': timestamp,
            'Metric': metric,
            'Violation': violation,
            'X_bar': x_bar,
            'Range': range_val,
            'Action': action
        }
        alarms.append(alarm)
    return alarms

# Step 6: Simulate and Monitor Process
def run_spc_system():
    # Simulate data
    data = simulate_data(N_SUBGROUPS, SAMPLE_SIZE, METRICS, NOMINAL, STD_DEV)
    
    # Corrective actions dictionary
    corrective_actions = {
        f"{metric} X̄ beyond 3-sigma limits": f"Check {metric.lower().replace('_', ' ')} calibration",
        f"{metric} R beyond control limits": f"Inspect {metric.lower().replace('_', ' ')} process variability",
        f"{metric} X̄ 2/3 points beyond 2-sigma": f"Review {metric.lower().replace('_', ' ')} equipment settings",
        f"{metric} X̄ 4/5 points beyond 1-sigma": f"Monitor {metric.lower().replace('_', ' ')} for trends",
        f"{metric} X̄ 8 points on one side": f"Adjust {metric.lower().replace('_', ' ')} process mean"
    }
    
    # Process each metric
    all_alarms = []
    for metric in METRICS:
        # Calculate control limits
        limits = calculate_control_limits(data, metric, SAMPLE_SIZE)
        
        # Detect violations
        violations = detect_violations(limits['x_bar'], limits['ranges'], limits, metric)
        
        # Trigger alarms
        alarms = trigger_alarm(violations, data, metric, limits, corrective_actions)
        all_alarms.extend(alarms)
    
    # Step 7: Output Results
    print("SPC System Results:")
    print("Timestamp           | Metric                  | Violation                          | X̄ Value | Range | Action")
    print("-" * 100)
    for alarm in all_alarms:
        print(f"{alarm['Timestamp']} | {alarm['Metric']:<22} | {alarm['Violation']:<33} | "
              f"{alarm['X_bar']:.2f} | {alarm['Range']:.2f} | {alarm['Action']}")
    
    # Step 8: Performance Metrics
    total_violations = len(all_alarms)
    variability = {metric: np.std(data[metric].apply(np.mean)) for metric in METRICS}
    mean_deviation = {metric: np.mean(np.abs(data[metric].apply(np.mean) - NOMINAL[metric]))
                      for metric in METRICS}
    
    print("\nPerformance Metrics:")
    print(f"Total Out-of-Control Alarms: {total_violations}")
    for metric in METRICS:
        print(f"{metric} Variability (Std Dev): {variability[metric]:.2f}")
        print(f"{metric} Mean Deviation from Nominal: {mean_deviation[metric]:.2f}")

# Run the SPC system
if __name__ == "__main__":
    run_spc_system()
