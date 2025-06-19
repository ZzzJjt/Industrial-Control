# main.py — NMPC controller for steam generator startup

import numpy as np
from scipy.integrate import solve_ivp
from scipy.optimize import minimize
from model import steam_generator_model
from controller import NMPCController

# Initial conditions: [Temperature, Pressure, FlowRate]
x0 = np.array([50.0, 1.0, 0.0])  # Initial temp (°C), pressure (MPa), flow (kg/s)
t_final = 600  # 10 minutes
dt = 10
times = np.arange(0, t_final, dt)
states = [x0]
controller = NMPCController(horizon=5, dt=dt)

for t in times[:-1]:
    x_current = states[-1]
    u_opt = controller.optimize_control(x_current)
    sol = solve_ivp(steam_generator_model, [0, dt], x_current, args=(u_opt,), t_eval=[dt])
    states.append(sol.y[:, -1])

states = np.array(states)

# Simple output
print("Final temperature:", states[-1, 0])
print("Final pressure:", states[-1, 1])
print("Final flow rate:", states[-1, 2])

# model.py — Nonlinear dynamic model of the steam generator

def steam_generator_model(t, x, u):
    T, P, F = x         # Temperature (°C), Pressure (MPa), Flow Rate (kg/s)
    Q, V = u            # Heat input (kW), valve position (0–1)

    # Simplified dynamics
    dTdt = (Q - 0.01 * (T - 100) - 0.05 * F) / 10.0
    dPdt = (0.01 * Q - 0.02 * V * P) / 5.0
    dFdt = (V * 10 - F) / 3.0

    return [dTdt, dPdt, dFdt]


  # controller.py — Nonlinear MPC controller logic

import numpy as np
from scipy.optimize import minimize
from model import steam_generator_model
from scipy.integrate import solve_ivp

class NMPCController:
    def __init__(self, horizon=10, dt=5):
        self.horizon = horizon
        self.dt = dt
        self.Q_target = 300.0  # Target temp
        self.P_target = 5.0    # Target pressure
        self.F_target = 10.0   # Target flow

    def objective(self, u_flat, x0):
        u_seq = u_flat.reshape(-1, 2)
        x = np.array(x0)
        cost = 0
        for u in u_seq:
            sol = solve_ivp(steam_generator_model, [0, self.dt], x, args=(u,), t_eval=[self.dt])
            x = sol.y[:, -1]
            T_err = (x[0] - self.Q_target) ** 2
            P_err = (x[1] - self.P_target) ** 2
            F_err = (x[2] - self.F_target) ** 2
            cost += T_err + P_err + F_err
        return cost

    def optimize_control(self, x0):
        u0 = np.array([200.0, 0.5] * self.horizon)  # Initial guess
        bounds = [(0, 500), (0, 1)] * self.horizon
        result = minimize(self.objective, u0, args=(x0,), bounds=bounds, method='SLSQP')
        return result.x[:2] if result.success else [200.0, 0.5]
