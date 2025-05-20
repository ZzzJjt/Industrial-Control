import numpy as np

class SteelQualitySimulator:
    def __init__(self):
        # Quality metric targets and standard deviations
        self.metrics = {
            'tensile_strength': {'target': 500.0, 'std': 5.0, 'unit': 'MPa'},
            'thickness': {'target': 2.0, 'std': 0.02, 'unit': 'mm'},
            'surface_finish': {'target': 1.5, 'std': 0.1, 'unit': 'µm'}
        }
        self.subgroup_size = 5
        self.drift_start = 500  # Sample index for drift
        self.drift_magnitude = {'tensile_strength': 10.0, 'thickness': 0.03, 'surface_finish': 0.15}

    def generate_subgroup(self, sample_idx):
        # Simulate a subgroup of measurements
        subgroup = {}
        for metric, params in self.metrics.items():
            mean = params['target']
            if sample_idx >= self.drift_start:
                mean += self.drift_magnitude[metric]  # Introduce drift
            data = np.random.normal(mean, params['std'], self.subgroup_size)
            subgroup[metric] = data
        return subgroup
