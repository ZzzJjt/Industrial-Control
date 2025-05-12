import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
from scipy import stats

class SteelQualitySPC:
    def __init__(self):
        # Process parameters for steel manufacturing
        self.target_thickness = 5.0  # mm
        self.target_tensile = 450    # MPa
        self.target_roughness = 1.2  # μm
        
        # Historical process data (for control limits calculation)
        self.historical_samples = {
            'thickness': self._generate_normal_data(5.0, 0.1, 100),
            'tensile': self._generate_normal_data(450, 15, 100),
            'roughness': self._generate_normal_data(1.2, 0.15, 100)
        }
        
        # Control limits (calculated from historical data)
        self.control_limits = {
            'thickness': self._calculate_control_limits(self.historical_samples['thickness']),
            'tensile': self._calculate_control_limits(self.historical_samples['tensile']),
            'roughness': self._calculate_control_limits(self.historical_samples['roughness'])
        }
        
        # Current production data
        self.current_samples = {
            'thickness': [],
            'tensile': [],
            'roughness': []
        }
        
        # Alarm tracking
        self.alarms = []
        self.sample_count = 0
        self.subgroup_size = 5  # Sample size for each subgroup
    
    def _generate_normal_data(self, mean, std, size):
        """Generate normally distributed process data"""
        return np.random.normal(mean, std, size)
    
    def _calculate_control_limits(self, data, subgroup_size=5):
        """Calculate X-bar and R chart control limits"""
        # Reshape data into subgroups
        subgroups = np.array(data[:len(data)//subgroup_size*subgroup_size]).reshape(-1, subgroup_size)
        
        # Calculate X-bar and R for each subgroup
        x_bars = np.mean(subgroups, axis=1)
        ranges = np.ptp(subgroups, axis=1)
        
        # Calculate control limits constants
        A2 = 0.577  # For subgroup size = 5
        D3 = 0      # For subgroup size = 5
        D4 = 2.114   # For subgroup size = 5
        
        # Calculate overall averages
        x_bar_avg = np.mean(x_bars)
        r_avg = np.mean(ranges)
        
        # Calculate control limits
        limits = {
            'x_bar': {
                'UCL': x_bar_avg + A2 * r_avg,
                'LCL': x_bar_avg - A2 * r_avg,
                'CL': x_bar_avg
            },
            'range': {
                'UCL': D4 * r_avg,
                'LCL': D3 * r_avg,
                'CL': r_avg
            }
        }
        
        return limits
    
    def generate_sample(self):
        """Simulate new production samples with occasional special causes"""
        self.sample_count += 1
        
        # Base process variation
        thickness = np.random.normal(self.target_thickness, 0.1)
        tensile = np.random.normal(self.target_tensile, 15)
        roughness = np.random.normal(self.target_roughness, 0.15)
        
        # Introduce special causes occasionally (15% chance)
        if np.random.random() < 0.15:
            cause = np.random.choice(['shift', 'trend', 'outlier'])
            
            if cause == 'shift':
                # Process mean shift
                shift_amount = np.random.uniform(1.5, 3.0)
                if np.random.random() < 0.5:
                    thickness += shift_amount * 0.1
                    tensile += shift_amount * 15
                else:
                    thickness -= shift_amount * 0.1
                    tensile -= shift_amount * 8
                roughness += shift_amount * 0.05
                
            elif cause == 'trend':
                # Progressive trend
                trend = self.sample_count % 20
                thickness += trend * 0.01
                tensile -= trend * 0.8
                roughness += trend * 0.005
                
            elif cause == 'outlier':
                # Single outlier
                if np.random.random() < 0.5:
                    thickness += np.random.uniform(2.0, 4.0) * 0.1
                else:
                    tensile += np.random.uniform(2.0, 4.0) * 15
                roughness += np.random.uniform(1.0, 2.0) * 0.1
        
        return {
            'thickness': max(3.0, min(7.0, thickness)),  # Physical limits
            'tensile': max(300, min(600, tensile)),       # Physical limits
            'roughness': max(0.5, min(3.0, roughness))    # Physical limits
        }
    
    def add_sample(self, sample):
        """Add new sample to current production data"""
        for key in self.current_samples:
            self.current_samples[key].append(sample[key])
        
        # Check for control violations when we have complete subgroups
        if len(self.current_samples['thickness']) % self.subgroup_size == 0:
            self._check_control_rules()
    
    def _check_control_rules(self):
        """Apply Western Electric rules to detect out-of-control conditions"""
        for parameter in self.current_samples:
            data = self.current_samples[parameter]
            if len(data) < self.subgroup_size:
                continue
                
            # Get the most recent subgroup
            subgroup = data[-self.subgroup_size:]
            x_bar = np.mean(subgroup)
            r = np.ptp(subgroup)
            
            limits = self.control_limits[parameter]
            
            # Rule 1: Point outside control limits
            if x_bar > limits['x_bar']['UCL'] or x_bar < limits['x_bar']['LCL']:
                self._trigger_alarm(parameter, f"Mean {x_bar:.2f} outside control limits")
            
            # Rule 2: 2 of 3 points beyond 2 sigma
            if len(data) >= 3 * self.subgroup_size:
                prev_points = [np.mean(data[i:i+self.subgroup_size]) 
                             for i in range(-3*self.subgroup_size, 0, self.subgroup_size)]
                sigma = (limits['x_bar']['UCL'] - limits['x_bar']['CL']) / 3
                beyond_2sigma = [abs(p - limits['x_bar']['CL']) > 2 * sigma for p in prev_points]
                if sum(beyond_2sigma) >= 2:
                    self._trigger_alarm(parameter, "2 of 3 points beyond 2σ limit")
            
            # Rule 3: 4 of 5 points beyond 1 sigma
            if len(data) >= 5 * self.subgroup_size:
                prev_points = [np.mean(data[i:i+self.subgroup_size]) 
                             for i in range(-5*self.subgroup_size, 0, self.subgroup_size)]
                beyond_1sigma = [abs(p - limits['x_bar']['CL']) > sigma for p in prev_points]
                if sum(beyond_1sigma) >= 4:
                    self._trigger_alarm(parameter, "4 of 5 points beyond 1σ limit")
            
            # Rule 4: 8 consecutive points on same side of center
            if len(data) >= 8 * self.subgroup_size:
                prev_points = [np.mean(data[i:i+self.subgroup_size]) 
                             for i in range(-8*self.subgroup_size, 0, self.subgroup_size)]
                if all(p > limits['x_bar']['CL'] for p in prev_points):
                    self._trigger_alarm(parameter, "8 consecutive points above center")
                elif all(p < limits['x_bar']['CL'] for p in prev_points):
                    self._trigger_alarm(parameter, "8 consecutive points below center")
    
    def _trigger_alarm(self, parameter, reason):
        """Record quality alarm and suggest corrective action"""
        alarm = {
            'parameter': parameter,
            'sample_number': self.sample_count,
            'reason': reason,
            'timestamp': pd.Timestamp.now(),
            'action': self._suggest_action(parameter, reason)
        }
        self.alarms.append(alarm)
        print(f"ALARM: {parameter} - {reason}")
        print(f"Suggested action: {alarm['action']}\n")
    
    def _suggest_action(self, parameter, reason):
        """Generate data-driven corrective action suggestions"""
        actions = {
            'thickness': {
                'high': "Check roller alignment and reduce rolling pressure",
                'low': "Increase rolling pressure and check material hardness",
                'trend': "Inspect roller wear and check temperature control",
                'variability': "Verify material consistency and check roller bearings"
            },
            'tensile': {
                'high': "Review alloy composition and check cooling rate",
                'low': "Increase carbon content and verify heat treatment",
                'trend': "Check furnace temperature stability",
                'variability': "Verify raw material quality and check process timing"
            },
            'roughness': {
                'high': "Check roller surface condition and lubrication",
                'low': "Verify finishing process parameters",
                'trend': "Inspect tool wear and surface treatment",
                'variability': "Check material uniformity and process consistency"
            }
        }
        
        if "outside control limits" in reason or "beyond" in reason:
            direction = 'high' if 'above' in reason or 'UCL' in reason else 'low'
            return actions[parameter][direction]
        elif "consecutive" in reason:
            return actions[parameter]['trend']
        else:
            return actions[parameter]['variability']
    
    def plot_control_charts(self):
        """Generate X-bar and R charts for all parameters"""
        fig, axes = plt.subplots(3, 2, figsize=(15, 12))
        
        for i, parameter in enumerate(['thickness', 'tensile', 'roughness']):
            if not self.current_samples[parameter]:
                continue
                
            # Prepare subgroup data
            subgroups = np.array(self.current_samples[parameter][:len(self.current_samples[parameter])//self.subgroup_size*self.subgroup_size])
            subgroups = subgroups.reshape(-1, self.subgroup_size)
            x_bars = np.mean(subgroups, axis=1)
            ranges = np.ptp(subgroups, axis=1)
            
            # X-bar chart
            ax = axes[i, 0]
            ax.plot(x_bars, 'b-o', label='X-bar')
            ax.axhline(self.control_limits[parameter]['x_bar']['UCL'], color='r', linestyle='--', label='UCL')
            ax.axhline(self.control_limits[parameter]['x_bar']['CL'], color='g', linestyle='-', label='CL')
            ax.axhline(self.control_limits[parameter]['x_bar']['LCL'], color='r', linestyle='--', label='LCL')
            ax.set_title(f'X-bar Chart for {parameter.capitalize()}')
            ax.set_ylabel(parameter)
            ax.set_xlabel('Subgroup Number')
            ax.legend()
            ax.grid(True)
            
            # R chart
            ax = axes[i, 1]
            ax.plot(ranges, 'b-o', label='Range')
            ax.axhline(self.control_limits[parameter]['range']['UCL'], color='r', linestyle='--', label='UCL')
            ax.axhline(self.control_limits[parameter]['range']['CL'], color='g', linestyle='-', label='CL')
            ax.axhline(self.control_limits[parameter]['range']['LCL'], color='r', linestyle='--', label='LCL')
            ax.set_title(f'R Chart for {parameter.capitalize()}')
            ax.set_ylabel(f'{parameter} Range')
            ax.set_xlabel('Subgroup Number')
            ax.legend()
            ax.grid(True)
        
        plt.tight_layout()
        plt.show()

def simulate_production(spc_system, num_samples=100):
    """Run production simulation with SPC monitoring"""
    print("Starting steel production simulation with SPC monitoring...\n")
    
    for _ in range(num_samples):
        sample = spc_system.generate_sample()
        spc_system.add_sample(sample)
    
    print("\nSimulation complete. Summary of quality alarms:")
    for alarm in spc_system.alarms:
        print(f"Sample {alarm['sample_number']}: {alarm['parameter']} - {alarm['reason']}")
    
    # Plot control charts
    spc_system.plot_control_charts()

if __name__ == "__main__":
    spc_system = SteelQualitySPC()
    simulate_production(spc_system, 200)
