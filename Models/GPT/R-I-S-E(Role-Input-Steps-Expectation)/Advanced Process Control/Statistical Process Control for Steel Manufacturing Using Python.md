import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from collections import deque

# --- Simulate Real-time Data Stream ---
def simulate_process_data(n=100):
    np.random.seed(42)
    data = pd.DataFrame({
        'TensileStrength': np.random.normal(400, 10, n),
        'Thickness': np.random.normal(5.0, 0.1, n),
        'SurfaceFinish': np.random.normal(1.5, 0.2, n)
    })
    return data

# --- Control Chart Calculations ---
def calculate_control_limits(data, subgroup_size):
    groups = [data[i:i+subgroup_size] for i in range(0, len(data), subgroup_size)]
    means = np.array([g.mean() for g in groups])
    ranges = np.array([g.max() - g.min() for g in groups])

    x_bar = means.mean(axis=0)
    r_bar = ranges.mean(axis=0)
    A2 = 0.577  # for n=5
    UCL_X = x_bar + A2 * r_bar
    LCL_X = x_bar - A2 * r_bar
    return x_bar, r_bar, UCL_X, LCL_X

# --- Alarm Logic ---
def check_control_limits(value, UCL, LCL):
    return value > UCL or value < LCL

def recommend_action(metric):
    actions = {
        'TensileStrength': 'Check furnace temperature and alloy composition.',
        'Thickness': 'Inspect roller settings and alignment.',
        'SurfaceFinish': 'Review lubrication and polishing parameters.'
    }
    return actions.get(metric, 'Review production settings.')

# --- Real-time Monitoring ---
def monitor_process(data, subgroup_size=5):
    alerts = []
    buffer = deque(maxlen=subgroup_size)

    for i in range(len(data)):
        buffer.append(data.iloc[i])
        if len(buffer) == subgroup_size:
            df_buffer = pd.DataFrame(buffer)
            for col in df_buffer.columns:
                x_bar, r_bar, UCL_X, LCL_X = calculate_control_limits(df_buffer[col], subgroup_size)
                current_mean = df_buffer[col].mean()

                if check_control_limits(current_mean, UCL_X, LCL_X):
                    alert = f"[ALERT] {col} out of control at index {i}. Suggestion: {recommend_action(col)}"
                    alerts.append(alert)
                    print(alert)
    return alerts

# --- Run Example ---
data = simulate_process_data(100)
alerts = monitor_process(data, subgroup_size=5)
