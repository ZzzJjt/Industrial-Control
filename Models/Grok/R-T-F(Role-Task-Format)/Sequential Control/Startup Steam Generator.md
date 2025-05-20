import casadi as ca
import numpy as np

class SteamGeneratorModel:
    def __init__(self):
        # Model parameters
        self.cp = 4.18  # Specific heat capacity (kJ/kg·K)
        self.hf = 1000.0  # Fuel enthalpy (kJ/kg)
        self.hs = 2800.0  # Steam enthalpy (kJ/kg)
        self.k1 = 0.5  # Pressure-temperature coefficient
        self.k2 = 0.01  # Pressure-mass coefficient
        self.Q = 100.0  # Heat loss (kW)
        
        # State variables: mass (m), temperature (T)
        self.nx = 2  # Number of states
        self.nu = 2  # Number of controls (fuel flow, feedwater flow)
        
        # Define symbolic variables
        self.x = ca.SX.sym('x', self.nx)  # States: [m, T]
        self.u = ca.SX.sym('u', self.nu)  # Controls: [u_f, u_w]
        self.dt = 30.0  # Time step (seconds)
        
        # Define dynamics
        m, T = self.x[0], self.x[1]
        u_f, u_w = self.u[0], self.u[1]
        F = 0.1 * m  # Simplified steam flow (proportional to mass)
        
        # Mass balance: dm/dt = u_w - F
        dm_dt = u_w - F
        # Energy balance: dT/dt = (u_f * hf - F * hs + Q) / (m * cp)
        dT_dt = (u_f * self.hf - F * self.hs + self.Q) / (m * self.cp)
        # Pressure: P = k1 * T + k2 * m^2
        P = self.k1 * T + self.k2 * m * m
        
        # State derivatives
        self.f = ca.vertcat(dm_dt, dT_dt)
        # Outputs: [P, T, F]
        self.h = ca.vertcat(P, T, F)
        
        # Create integrator
        self.integrator = ca.integrator('F', 'cvodes', 
                                       {'x': self.x, 'u': self.u, 'ode': self.f}, 
                                       {'t0': 0, 'tf': self.dt})
    
    def simulate(self, x0, u):
        # Simulate one step
        res = self.integrator(x0=x0, u=u)
        x_next = res['xf']
        P, T, F = ca.Function('h', [self.x], [self.h])(x_next)
        return x_next, P, T, F
