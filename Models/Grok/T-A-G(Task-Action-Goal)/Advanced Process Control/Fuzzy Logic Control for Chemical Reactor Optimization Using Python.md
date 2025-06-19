import numpy as np

class ReactorModel:
    def __init__(self):
        # Reactor parameters
        self.m = 1000.0  # Mass (kg)
        self.cp = 4.18  # Specific heat capacity (kJ/kg·K)
        self.delta_Hr = -50000.0  # Reaction enthalpy (kJ/kmol)
        self.k0 = 1e6  # Pre-exponential factor (1/s)
        self.Ea = 50000.0  # Activation energy (J/mol)
        self.R = 8.314  # Gas constant (J/mol·K)
        self.Q_loss = 1000.0  # Constant heat loss (kW)
        self.dt = 1.0  # Time step (s)

    def reaction_rate(self, T):
        # Arrhenius reaction rate: k0 * exp(-Ea/RT)
        return self.k0 * np.exp(-self.Ea / (self.R * T))

    def simulate(self, T, Q_in, disturbance):
        # Energy balance: dT/dt = (Q_in - Q_loss + delta_Hr * R(T)) / (m * cp) + disturbance
        dT_dt = (Q_in - self.Q_loss + self.delta_Hr * self.reaction_rate(T)) / (self.m * self.cp) + disturbance
        T_next = T + dT_dt * self.dt
        return T_next
