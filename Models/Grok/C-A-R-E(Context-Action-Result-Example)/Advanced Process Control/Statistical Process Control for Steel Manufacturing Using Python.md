import numpy as np
from scipy import stats

# Steel Rolling Process Simulation
class SteelProcess:
    def __init__(self, dt=60.0):
        self.dt = dt  # Sampling interval (s)
        self.tensile_mean = 500.0  # MPa
        self.thickness_mean = 2.0  # mm
        self.finish_mean = 1.0     # µm
        self.tensile_std = 5.0     # MPa
        self.thickness_std = 0.05  # mm
        self.finish_std = 0.1      # µm
        self.thickness_drift = 0.01 / 60.0  # mm/s (0.01 mm/min)
        self.time = 0.0

    def sample(self, n_samples=5):
        """Generate subgroup samples with thickness drift"""
        tensile = np.random.normal(self.tensile_mean, self.tensile_std, n_samples)
        thickness = np.random.normal(self.thickness_mean + self.thickness_drift * self.time,
                                   self.thickness_std, n_samples)
        finish = np.random.normal(self.finish_mean, self.finish_std, n_samples)
        self.time += self.dt
        return tensile, thickness, finish

# SPC System
class SPC:
    def __init__(self, n_samples=5):
        self.n_samples = n_samples
        self.tensile_stats = {'xbar': [], 'R': [], 'S': [], 'xbar_mean': 0.0,
                             'R_mean': 0.0, 'S_mean': 0.0, 'xbar_UCL': 0.0,
                             'xbar_LCL': 0.0, 'R_UCL': 0.0, 'S_UCL': 0.0}
        self.thickness_stats = {'xbar': [], 'R': [], 'S': [], 'xbar_mean': 0.0,
                               'R_mean': 0.0, 'S_mean': 0.0, 'xbar_UCL': 0.0,
                               'xbar_LCL': 0.0, 'R_UCL': 0.0, 'S_UCL': 0.0}
        self.finish_stats = {'xbar': [], 'R': [], 'S': [], 'xbar_mean': 0.0,
                            'R_mean': 0.0, 'S_mean': 0.0, 'xbar_UCL': 0.0,
                            'xbar_LCL': 0.0, 'R_UCL': 0.0, 'S_UCL': 0.0}
        self.alarms = []
        self.actions = []

    def initialize_charts(self, initial_data):
        """Calculate control limits from initial stable data"""
        for metric, data, stats in [('Tensile', initial_data[0], self.tensile_stats),
                                   ('Thickness', initial_data[1], self.thickness_stats),
                                   ('Finish', initial_data[2], self.finish_stats)]:
            xbar = [np.mean(subgroup) for subgroup in data]
            R = [np.ptp(subgroup) for subgroup in data]
            S = [np.std(subgroup, ddof=1) for subgroup in data]
            stats['xbar_mean'] = np.mean(xbar)
            stats['R_mean'] = np.mean(R)
            stats['S_mean'] = np.mean(S)
            # Control limits (n=5, A2=0.577, D4=2.114, D3=0, c4=0.951)
            stats['xbar_UCL'] = stats['xbar_mean'] + 0.577 * stats['R_mean']
            stats['xbar_LCL'] = stats['xbar_mean'] - 0.577 * stats['R_mean']
            stats['R_UCL'] = 2.114 * stats['R_mean']
            stats['R_LCL'] = 0.0
            stats['S_UCL'] = stats['S_mean'] / 0.951 * stats.norm.ppf(0.99865)
            stats['S_LCL'] = stats['S_mean'] / 0.951 * stats.norm.ppf(0.00135)

    def update_charts(self, subgroup_data, t):
        """Update control charts and check for out-of-control conditions"""
        for metric, data, stats in [('Tensile', subgroup_data[0], self.tensile_stats),
                                   ('Thickness', subgroup_data[1], self.thickness_stats),
                                   ('Finish', subgroup_data[2], self.finish_stats)]:
            xbar = np.mean(data)
            R = np.ptp(data)
            S = np.std(data, ddof=1)
            stats['xbar'].append(xbar)
            stats['R'].append(R)
            stats['S'].append(S)
            
            # Western Electric Rules
            alarm = None
            # Rule 1: Point beyond 3σ
            if xbar > stats['xbar_UCL'] or xbar < stats['xbar_LCL']:
                alarm = f"{metric}: Point beyond 3σ at t={t}s"
            elif R > stats['R_UCL'] or S > stats['S_UCL']:
                alarm = f"{metric}: Variability beyond limits at t={t}s"
            # Rule 2: 2/3 points beyond 2σ
            elif len(stats['xbar']) >= 3:
                sigma_2 = stats['R_mean'] * 0.577 * 2 / 3
                recent = stats['xbar'][-3:]
                if sum(1 for x in recent if x > stats['xbar_mean'] + sigma_2) >= 2 or \
                   sum(1 for x in recent if x < stats['xbar_mean'] - sigma_2) >= 2:
                    alarm = f"{metric}: 2/3 points beyond 2σ at t={t}s"
            # Rule 3: 4/5 points beyond 1σ
            elif len(stats['xbar']) >= 5:
                sigma_1 = stats['R_mean'] * 0.577 / 3
                recent = stats['xbar'][-5:]
                if sum(1 for x in recent if x > stats['xbar_mean'] + sigma_1) >= 4 or \
                   sum(1 for x in recent if x < stats['xbar_mean'] - sigma_1) >= 4:
                    alarm = f"{metric}: 4/5 points beyond 1σ at t={t}s"
            # Rule 4: 8 points on one side
            elif len(stats['xbar']) >= 8:
                recent = stats['xbar'][-8:]
                if all(x > stats['xbar_mean'] for x in recent) or \
                   all(x < stats['xbar_mean'] for x in recent):
                    alarm = f"{metric}: 8 points on one side at t={t}s"
            
            if alarm:
                self.alarms.append(alarm)
                # Suggest corrective actions
                if metric == 'Thickness':
                    delta = xbar - stats['xbar_mean']
                    action = f"Adjust rolling pressure by {-delta*100:.1f} kN"
                elif metric == 'Tensile':
                    delta = xbar - stats['xbar_mean']
                    action = f"Adjust cooling rate by {-delta*0.1:.1f} °C/s"
                else:  # Finish
                    delta = xbar - stats['xbar_mean']
                    action = f"Adjust polishing speed by {-delta*0.1:.1f} m/s"
                self.actions.append(f"{metric} at t={t}s: {action}")

# Baseline (Threshold-Based)
class Baseline:
    def __init__(self):
        self.spec_limits = {
            'Tensile': (485.0, 515.0),  # MPa
            'Thickness': (1.9, 2.1),    # mm
            'Finish': (0.8, 1.2)        # µm
        }
        self.alarms = []
        self.actions = []

    def check(self, subgroup_data, t):
        """Check metrics against specification limits"""
        for metric, data, limits in [('Tensile', subgroup_data[0], self.spec_limits['Tensile']),
                                   ('Thickness', subgroup_data[1], self.spec_limits['Thickness']),
                                   ('Finish', subgroup_data[2], self.spec_limits['Finish'])]:
            xbar = np.mean(data)
            if xbar < limits[0] or xbar > limits[1]:
                self.alarms.append(f"{metric}: Out of spec ({xbar:.2f}) at t={t}s")
                # Same corrective actions as SPC
                if metric == 'Thickness':
                    delta = xbar - 2.0
                    action = f"Adjust rolling pressure by {-delta*100:.1f} kN"
                elif metric == 'Tensile':
                    delta = xbar - 500.0
                    action = f"Adjust cooling rate by {-delta*0.1:.1f} °C/s"
                else:
                    delta = xbar - 1.0
                    action = f"Adjust polishing speed by {-delta*0.1:.1f} m/s"
                self.actions.append(f"{metric} at t={t}s: {action}")

# Simulate Process
def simulate(process, spc, baseline, t_sim=3600.0, dt=60.0):
    n_steps = int(t_sim / dt)
    t = np.linspace(0, t_sim, n_steps)
    
    # Generate initial stable data (first 10 subgroups)
    initial_data = [[], [], []]
    process.time = 0.0
    for _ in range(10):
        tensile, thickness, finish = process.sample(n_samples=5)
        initial_data[0].append(tensile)
        initial_data[1].append(thickness)
        initial_data[2].append(finish)
    spc.initialize_charts(initial_data)
    
    # Reset time and simulate with drift
    process.time = 0.0
    for i in range(n_steps):
        tensile, thickness, finish = process.sample(n_samples=5)
        spc.update_charts((tensile, thickness, finish), t[i])
        baseline.check((tensile, thickness, finish), t[i])
    
    # Calculate performance metrics
    mse_thickness_spc = np.mean((np.array(spc.thickness_stats['xbar']) - 2.0)**2)
    mse_thickness_base = np.mean((np.array(baseline.thickness_stats['xbar']) - 2.0)**2)
    alarms_spc = len(spc.alarms)
    alarms_base = len(baseline.alarms)
    
    print(f"SPC Thickness MSE: {mse_thickness_spc:.4f} mm²")
    print(f"Baseline Thickness MSE: {mse_thickness_base:.4f} mm²")
    print(f"SPC Alarms: {alarms_spc}")
    print(f"Baseline Alarms: {alarms_base}")
    
    # Store results
    results = {
        'time': t,
        'tensile_xbar': spc.tensile_stats['xbar'],
        'thickness_xbar': spc.thickness_stats['xbar'],
        'finish_xbar': spc.finish_stats['xbar'],
        'tensile_R': spc.tensile_stats['R'],
        'thickness_R': spc.thickness_stats['R'],
        'finish_R': spc.finish_stats['R'],
        'tensile_S': spc.tensile_stats['S'],
        'thickness_S': spc.thickness_stats['S'],
        'finish_S': spc.finish_stats['S'],
        'spc_alarms': spc.alarms,
        'spc_actions': spc.actions,
        'baseline_alarms': baseline.alarms,
        'baseline_actions': baseline.actions
    }
    return results

# Main Execution
if __name__ == "__main__":
    process = SteelProcess(dt=60.0)
    spc = SPC(n_samples=5)
    baseline = Baseline()
    results = simulate(process, spc, baseline, t_sim=3600.0, dt=60.0)
