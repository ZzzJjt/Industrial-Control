# Install GEKKO before running: pip install gekko
from gekko import GEKKO
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Initialize model
m = GEKKO(remote=False)
m.time = np.linspace(0, 60, 61)  # 60-minute time horizon

# Constants
T_max = 520        # °C
P_min = 30         # bar
Flow_max = 1000    # kg/h

# Control variable
Q = m.MV(value=0, lb=0, ub=1)  # Heat input (normalized)
Q.STATUS = 1
Q.DCOST = 0.01

# State variables
T = m.SV(value=25)   # Temperature (°C)
P = m.SV(value=1)    # Pressure (bar)
F = m.SV(value=100)  # Flow (kg/h)

# Dynamics (simplified non-linear model)
m.Equation(T.dt() == 5 * Q - 0.01 * (T - 25))
m.Equation(P.dt() == 0.2 * Q * (1 + 0.01 * T) - 0.05 * P)
m.Equation(F.dt() == 10 * Q - 0.1 * F)

# Constraints
m.Equation(T <= T_max)
m.Equation(P >= P_min)
m.Equation(F <= Flow_max)

# Objective: reach target conditions
m.Obj((T - 500)**2 + (P - 100)**2 + 0.1 * Q**2)

# Solve with MPC
m.options.IMODE = 6
m.solve(disp=True)

# Plot results
plt.plot(m.time, T.value, label='Temperature (°C)')
plt.plot(m.time, P.value, label='Pressure (bar)')
plt.plot(m.time, F.value, label='Flow (kg/h)')
plt.plot(m.time, Q.value, label='Heat Input Q')
plt.xlabel('Time (min)')
plt.ylabel('Values')
plt.title('Steam Generator NMPC Startup')
plt.legend()
plt.grid()
plt.show()
