import numpy as np
import matplotlib.pyplot as plt
from scipy.stats import norm

# Step 1: Simulate Key Quality Metrics
def simulate_quality_metrics(num_samples, num_subgroups):
    """
    Simulate quality metrics for steel manufacturing.
    
    Parameters:
        num_samples (int): Number of samples per subgroup.
        num_subgroups (int): Number of subgroups.
    
    Returns:
        numpy.ndarray: Simulated quality metrics [subgroup_index, sample_index].
    """
    # Mean and standard deviation for each metric
    mean_strength = 500  # Mean tensile strength (MPa)
    std_strength = 20    # Std dev tensile strength (MPa)
    mean_thickness = 10  # Mean thickness (mm)
    std_thickness = 0.1  # Std dev thickness (mm)
    mean_surface_finish = 1.5  # Mean surface finish (Ra, microns)
    std_surface_finish = 0.05  # Std dev surface finish (microns)
    
    # Simulate data
    strength_data = np.random.normal(mean_strength, std_strength, (num_subgroups, num_samples))
    thickness_data = np.random.normal(mean_thickness, std_thickness, (num_subgroups, num_samples))
    surface_finish_data = np.random.normal(mean_surface_finish, std_surface_finish, (num_subgroups, num_samples))
    
    return strength_data, thickness_data, surface_finish_data

# Step 2: Implement SPC Tools
class SPCTools:
    def __init__(self, data):
        self.data = data
        self.num_subgroups = data.shape[0]
        self.num_samples = data.shape[1]
        self.x_bar = np.mean(data, axis=1)
        self.r = np.max(data, axis=1) - np.min(data, axis=1)
        self.x_double_bar = np.mean(self.x_bar)
        self.r_bar = np.mean(self.r)
        
        # Control chart constants
        if self.num_samples == 2:
            A2 = 1.880
            D3 = 0.0
            D4 = 3.267
        elif self.num_samples == 3:
            A2 = 1.023
            D3 = 0.0
            D4 = 2.574
        elif self.num_samples == 4:
            A2 = 0.729
            D3 = 0.0
            D4 = 2.282
        elif self.num_samples == 5:
            A2 = 0.577
            D3 = 0.0
            D4 = 2.114
        else:
            raise ValueError("Number of samples per subgroup must be 2, 3, 4, or 5.")
        
        self.UCL_x_bar = self.x_double_bar + A2 * self.r_bar
        self.LCL_x_bar = self.x_double_bar - A2 * self.r_bar
        self.UCL_r = D4 * self.r_bar
        self.LCL_r = D3 * self.r_bar
        
    def plot_x_bar_chart(self):
        """
        Plot X̄ chart.
        """
        plt.figure(figsize=(10, 5))
        plt.plot(range(1, self.num_subgroups + 1), self.x_bar, marker='o', label='X̄')
        plt.axhline(y=self.x_double_bar, color='r', linestyle='--', label='X̄̄')
        plt.axhline(y=self.UCL_x_bar, color='g', linestyle='--', label='UCL')
        plt.axhline(y=self.LCL_x_bar, color='g', linestyle='--', label='LCL')
        plt.title('X̄ Chart')
        plt.xlabel('Subgroup Index')
        plt.ylabel('Mean Value')
        plt.legend()
        plt.grid(True)
        plt.show()
    
    def plot_r_chart(self):
        """
        Plot R chart.
        """
        plt.figure(figsize=(10, 5))
        plt.plot(range(1, self.num_subgroups + 1), self.r, marker='o', label='R')
        plt.axhline(y=self.r_bar, color='r', linestyle='--', label='R̄')
        plt.axhline(y=self.UCL_r, color='g', linestyle='--', label='UCL')
        plt.axhline(y=self.LCL_r, color='g', linestyle='--', label='LCL')
        plt.title('R Chart')
        plt.xlabel('Subgroup Index')
        plt.ylabel('Range')
        plt.legend()
        plt.grid(True)
        plt.show()
    
    def check_control_limits(self):
        """
        Check if any points are out of control limits.
        
        Returns:
            dict: Dictionary with out-of-control information.
        """
        out_of_control = {}
        
        # Check X̄ chart
        x_bar_out_of_control = []
        for i in range(self.num_subgroups):
            if self.x_bar[i] > self.UCL_x_bar or self.x_bar[i] < self.LCL_x_bar:
                x_bar_out_of_control.append(i + 1)
        if x_bar_out_of_control:
            out_of_control['X̄'] = x_bar_out_of_control
        
        # Check R chart
        r_out_of_control = []
        for i in range(self.num_subgroups):
            if self.r[i] > self.UCL_r or self.r[i] < self.LCL_r:
                r_out_of_control.append(i + 1)
        if r_out_of_control:
            out_of_control['R'] = r_out_of_control
        
        return out_of_control

# Step 3: Recommend Corrective Actions
def recommend_corrective_actions(out_of_control_info):
    """
    Recommend corrective actions based on out-of-control information.
    
    Parameters:
        out_of_control_info (dict): Dictionary with out-of-control information.
    
    Returns:
        str: Recommended corrective actions.
    """
    recommendations = ""
    if 'X̄' in out_of_control_info:
        recommendations += f"Subgroups {out_of_control_info['X̄']} have means out of control limits.\n"
        recommendations += "Investigate potential causes such as equipment malfunctions or material issues.\n"
    if 'R' in out_of_control_info:
        recommendations += f"Subgroups {out_of_control_info['R']} have ranges out of control limits.\n"
        recommendations += "Check for variability in raw materials or inconsistencies in the production process.\n"
    if not out_of_control_info:
        recommendations += "All subgroups are within control limits. No immediate corrective action required."
    return recommendations

# Simulation parameters
num_samples = 5  # Number of samples per subgroup
num_subgroups = 20  # Number of subgroups

# Simulate quality metrics
strength_data, thickness_data, surface_finish_data = simulate_quality_metrics(num_samples, num_subgroups)

# Analyze tensile strength using SPC tools
spc_strength = SPCTools(strength_data)
spc_strength.plot_x_bar_chart()
spc_strength.plot_r_chart()
out_of_control_strength = spc_strength.check_control_limits()

# Analyze thickness using SPC tools
spc_thickness = SPCTools(thickness_data)
spc_thickness.plot_x_bar_chart()
spc_thickness.plot_r_chart()
out_of_control_thickness = spc_thickness.check_control_limits()

# Analyze surface finish using SPC tools
spc_surface_finish = SPCTools(surface_finish_data)
spc_surface_finish.plot_x_bar_chart()
spc_surface_finish.plot_r_chart()
out_of_control_surface_finish = spc_surface_finish.check_control_limits()

# Recommend corrective actions
recommendations_strength = recommend_corrective_actions(out_of_control_strength)
recommendations_thickness = recommend_corrective_actions(out_of_control_thickness)
recommendations_surface_finish = recommend_corrective_actions(out_of_control_surface_finish)

# Output results
print("Tensile Strength Recommendations:")
print(recommendations_strength)
print("\nThickness Recommendations:")
print(recommendations_thickness)
print("\nSurface Finish Recommendations:")
print(recommendations_surface_finish)



