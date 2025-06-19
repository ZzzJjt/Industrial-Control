import numpy as np

class ReactorModel:
    def __init__(self):
        # Reactor parameters
        self.m = 1000.0  # Mass (kg)
        self.cp = 4.18  # Specific heat capacity (kJ/kg·K)
        self.delta_Hr = -50000.0  # Reaction enthalpy (kJ/kmol)
        self.k0 = 1e6  # Pre-exponential factor (1/s)
        self.Ea = 50000.0  # Activation energy (J/mol)
        self.Rg = 8.314  # Gas constant (J/mol·K)
        self.Q_loss = 1000.0  # Heat loss (kW)
        self.V = 10.0  # Reactor volume (m³)
        self.k_P = 0.1  # Vent loss coefficient (1/s)
        self.P_atm = 1.0  # Atmospheric pressure (bar)
        self.C = 1.0  # Reactant concentration (mol/m³, constant)
        self.dt = 1.0  # Time step (s)

    def reaction_rate(self, T):
        # Arrhenius rate: k0 * exp(-Ea/(Rg*T)) * C
        return self.k0 * np.exp(-self.Ea / (self.Rg * T)) * self.C

    def simulate(self, T, P, Q_in, u_P, d_T, d_P):
        # Temperature dynamics
        R = self.reaction_rate(T)
        dT_dt = (Q_in - self.Q_loss + self.delta_Hr * R) / (self.m * self.cp) + d_T
        
        # Pressure dynamics
        dP_dt = (self.Rg * T / self.V) * R - self.k_P * (P - self.P_atm) * u_P + d_P
        
        # Update states
        T_next = T + dT_dt * self.dt
        P_next = P + dP_dt * self.dt
        return T_next, P_next
