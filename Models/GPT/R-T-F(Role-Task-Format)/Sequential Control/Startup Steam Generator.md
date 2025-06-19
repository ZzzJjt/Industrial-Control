from gekko import GEKKO
import numpy as np
import matplotlib.pyplot as plt

# Time horizon
n_horizon = 20
dt = 1.0  # Time step in minutes

# Initialize GEKKO model
m = GEKKO(remote=False)
m.time = np.linspace(0, (n_horizon-1)*dt, n_horizon)

# Variables
P = m.Var(value=50, lb=30, ub=160)   # Pressure [bar]
T = m.Var(value=100, lb=100, ub=520) # Temperature [°C]
F = m.Var(value=10, lb=5, ub=100)    # Flow rate [kg/s]

# Manipulated variables
u_heater = m.MV(value=0.5, lb=0, ub=1)
u_pump = m.MV(value=0.5, lb=0, ub=1)
u_heater.STATUS = 1
u_pump.STATUS = 1

# Setpoints
target_P = 150  # bar
target_T = 500  # °C

# Dynamics (simplified)
m.Equation(P.dt() == -0.1*P + 0.5*u_pump + 0.3*u_heater)
m.Equation(T.dt() == -0.2*T + 1.0*u_heater)
m.Equation(F.dt() == -0.05*F + 0.4*u_pump)

# Objective: track setpoints and minimize control effort
m.Obj((P - target_P)**2)
m.Obj((T - target_T)**2)
m.Obj(0.1*u_heater**2 + 0.1*u_pump**2)

# Solver settings
m.options.IMODE = 6  # MPC
m.options.SOLVER = 3
m.solve(disp=True)

# Plot results
plt.figure(figsize=(10, 6))
plt.subplot(3,1,1)
plt.plot(m.time, P.value, label='Pressure [bar]')
plt.axhline(target_P, linestyle='--', color='gray')
plt.legend()

plt.subplot(3,1,2)
plt.plot(m.time, T.value, label='Temperature [°C]')
plt.axhline(target_T, linestyle='--', color='gray')
plt.legend()

plt.subplot(3,1,3)
plt.plot(m.time, F.value, label='Flow Rate [kg/s]')
plt.legend()

plt.tight_layout()
plt.show()
