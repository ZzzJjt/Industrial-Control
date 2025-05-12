import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Set random seed for reproducibility
np.random.seed(42)

# Process Parameters
n_samples = 100  # Number of sample groups
subgroup_size = 5  # Measurements per subgroup
metrics = {
    'tensile_strength': {'mean': 500, 'std': 10, 'lcl': 470, 'ucl': 530},  # MPa
    'thickness': {'mean': 2.0, 'std': 0.05, 'lcl': 1.85, 'ucl': 2.15},     # mm
    'surface_finish': {'mean': 0.5, 'std': 0.02, 'lcl': 0.44, 'ucl': 0.56}  # Ra (micrometers)
}
A2 = 0.577  # Control chart constant for subgroup size 5
D3 = 0.0    # Lower control limit factor for R chart
D4 = 2.114  # Upper control limit factor for R chart

# Simulate Steel Manufacturing Process
class SteelProcess:
    def __init__(self, metrics):
        self.metrics = metrics
        self.data = {key: [] for key in metrics}
        self.rng = np.random.default_rng(42)
    
    def generate_data(self, n_samples, subgroup_size, drift=False):
        for metric, params in self.metrics.items():
            data = []
            for i in range(n_samples):
                # Introduce drift for demonstration (e.g., gradual increase in mean)
                mean = params['mean'] + (0.02 * i if drift and i > n_samples // 2 else 0)
                subgroup = self.rng.normal(mean, params['std'], subgroup_size)
                data.append(subgroup)
            self.data[metric] = np.array(data)
    
    def get_subgroup_stats(self, metric):
        data = self.data[metric]
        X_bar = np.mean(data, axis=1)  # Subgroup means
        R = np.ptp(data, axis=1)       # Subgroup ranges
        return X_bar, R

# SPC Control Charts
class SPCController:
    def __init__(self, metrics, subgroup_size):
        self.metrics = metrics
        self.subgroup_size = subgroup_size
        self.control_limits = {}
        self.alarms = {key: [] for key in metrics}
        self.suggestions = {key: [] for key in metrics}
    
    def calculate_control_limits(self, metric, X_bar, R):
        X_bar_mean = np.mean(X_bar[:20])  # Use first 20 subgroups for baseline
        R_mean = np.mean(R[:20])
        
        # X̄ chart limits
        ucl_x = X_bar_mean + A2 * R_mean
        lcl_x = X_bar_mean - A2 * R_mean
        
        # R chart limits
        ucl_r = D4 * R_mean
        lcl_r = D3 * R_mean
        
        self.control_limits[metric] = {
            'x_bar': {'mean': X_bar_mean, 'ucl': ucl_x, 'lcl': lcl_x},
            'r': {'mean': R_mean, 'ucl': ucl_r, 'lcl': lcl_r}
        }
    
    def detect_violations(self, metric, X_bar, R):
        alarms = []
        suggestions = []
        x_limits = self.control_limits[metric]['x_bar']
        r_limits = self.control_limits[metric]['r']
        
        for i in range(len(X_bar)):
            alarm = None
            suggestion = None
            
            # Rule 1: Point beyond control limits
            if X_bar[i] > x_limits['ucl'] or X_bar[i] < x_limits['lcl']:
                alarm = f"Point {i} out of X̄ control limits"
                suggestion = "Check raw material quality or process settings"
            elif R[i] > r_limits['ucl'] or R[i] < r_limits['lcl']:
                alarm = f"Point {i} out of R control limits"
                suggestion = "Investigate equipment consistency or operator variability"
            
            # Rule 2: 7 points on one side of centerline
            if i >= 7:
                if all(X_bar[i-6:i+1] > x_limits['mean']):
                    alarm = f"Run of 7 points above centerline at {i}"
                    suggestion = "Adjust process to reduce mean shift"
                elif all(X_bar[i-6:i+1] < x_limits['mean']):
                    alarm = f"Run of 7 points below centerline at {i}"
                    suggestion = "Adjust process to increase mean"
            
            # Rule 3: Trend (7 points increasing or decreasing)
            if i >= 7:
                diffs = np.diff(X_bar[i-6:i+1])
                if all(diffs > 0):
                    alarm = f"Increasing trend at {i}"
                    suggestion = "Check for gradual process drift (e.g., tool wear)"
                elif all(diffs < 0):
                    alarm = f"Decreasing trend at {i}"
                    suggestion = "Check for material or environmental changes"
            
            alarms.append(alarm)
            suggestions.append(suggestion)
        
        self.alarms[metric] = alarms
        self.suggestions[metric] = suggestions
    
    def analyze_process(self, metric, X_bar, R):
        self.calculate_control_limits(metric, X_bar, R)
        self.detect_violations(metric, X_bar, R)
        
        # Count violations
        violations = sum(1 for a in self.alarms[metric] if a is not None)
        return violations

# Simulation and Analysis
def run_simulation():
    process = SteelProcess(metrics)
    process.generate_data(n_samples, subgroup_size, drift=True)
    spc = SPCController(metrics, subgroup_size)
    
    # Analyze each metric
    results = {}
    for metric in metrics:
        X_bar, R = process.get_subgroup_stats(metric)
        violations = spc.analyze_process(metric, X_bar, R)
        results[metric] = {
            'X_bar': X_bar,
            'R': R,
            'violations': violations,
            'control_limits': spc.control_limits[metric]
        }
    
    # Output results
    print("SPC Analysis Results:")
    for metric in metrics:
        print(f"\n{metric.replace('_', ' ').title()}:")
        print(f"Violations Detected: {results[metric]['violations']}")
        print(f"Mean X̄: {results[metric]['control_limits']['x_bar']['mean']:.2f}")
        print(f"Mean R: {results[metric]['control_limits']['r']['mean']:.2f}")
        alarms = [(i, a, s) for i, (a, s) in enumerate(zip(spc.alarms[metric], spc.suggestions[metric])) if a]
        if alarms:
            print("Alarms and Suggestions:")
            for i, alarm, suggestion in alarms[:3]:  # Limit to 3 for brevity
                print(f"Sample {i}: {alarm} -> {suggestion}")
    
    # Calculate overall quality metrics
    defect_rate = sum(r['violations'] for r in results.values()) / (n_samples * len(metrics)) * 100
    print(f"\nOverall Defect Rate: {defect_rate:.2f}%")
    
    # Visualization
    plt.figure(figsize=(12, 8))
    for i, metric in enumerate(metrics, 1):
        X_bar = results[metric]['X_bar']
        R = results[metric]['R']
        x_limits = results[metric]['control_limits']['x_bar']
        r_limits = results[metric]['control_limits']['r']
        
        plt.subplot(len(metrics), 2, 2*i-1)
        plt.plot(X_bar, 'b.-', label='X̄')
        plt.axhline(x_limits['mean'], color='k', linestyle='--', label='Centerline')
        plt.axhline(x_limits['ucl'], color='r', linestyle='--', label='UCL/LCL')
        plt.axhline(x_limits['lcl'], color='r', linestyle='--')
        plt.title(f"{metric.replace('_', ' ').title()} X̄ Chart")
        plt.xlabel('Sample')
        plt.ylabel(metric.replace('_', ' '))
        plt.legend()
        plt.grid(True)
        
        plt.subplot(len(metrics), 2, 2*i)
        plt.plot(R, 'b.-', label='R')
        plt.axhline(r_limits['mean'], color='k', linestyle='--', label='Centerline')
        plt.axhline(r_limits['ucl'], color='r', linestyle='--', label='UCL/LCL')
        plt.axhline(r_limits['lcl'], color='r', linestyle='--')
        plt.title(f"{metric.replace('_', ' ').title()} R Chart")
        plt.xlabel('Sample')
        plt.ylabel('Range')
        plt.legend()
        plt.grid(True)
    
    plt.tight_layout()
    plt.savefig('steel_spc_charts.png')

if __name__ == "__main__":
    run_simulation()
