import numpy as np
import casadi as ca
import matplotlib.pyplot as plt

# Simulation parameters
dt = 1.0        # Time step [s]
N = 20          # Prediction horizon
T_sim = 100     # Simulation time steps

# System parameters (simplified turbine model)
Cp = 1.0        # Heat capacity
tau = 10.0      # Time constant [s]
T_max = 600.0   # Max turbine temperature [°C]
T_min = 300.0   # Min turbine temperature [°C]
P_max = 100.0   # Max power output [MW]
Ramp_limit = 5.0  # Max ramp rate [MW/s]

# Load demand profile (step changes)
load_demand = np.concatenate([np.ones(30)*40, np.ones(40)*70, np.ones(30)*50])

# MPC model (states: T - temperature, P - power)
opti = ca.Opti()
T = opti.variable(1, N+1)
P = opti.variable(1, N+1)
Q = opti.variable(1, N)    # control input: fuel heat flow rate

# Cost function
cost = 0
for k in range(N):
    cost += 10 * (P[0, k] - load_demand[k])**2 + 0.1 * Q[0, k]**2
opti.minimize(cost)

# Dynamics (Euler discretization)
for k in range(N):
    dT = (-T[0, k] + Q[0, k]/Cp) / tau
    P_est = 0.5 * (T[0, k] - T_min)  # linear mapping to power output
    opti.subject_to(T[0, k+1] == T[0, k] + dT * dt)
    opti.subject_to(P[0, k] == P_est)

# Initial condition
T0 = 350.0
opti.subject_to(T[0, 0] == T0)

# Constraints
opti.subject_to(opti.bounded(T_min, T, T_max))
opti.subject_to(opti.bounded(0, Q, 500))
opti.subject_to(opti.bounded(0, P, P_max))
for k in range(N):
    opti.subject_to(ca.fabs(P[0, k+1] - P[0, k]) <= Ramp_limit)

# Solver settings
opti.solver("ipopt")
T_history = [T0]
P_history = []
Q_history = []

for t in range(T_sim):
    # Update demand in the cost function
    for k in range(N):
        if t+k < T_sim:
            load_d = load_demand[t+k]
        else:
            load_d = load_demand[-1]
        opti.set_value(load_d, load_d)

    # Solve MPC
    try:
        sol = opti.solve()
        Q_opt = sol.value(Q[0, 0])
        T_next = T_history[-1] + (-T_history[-1] + Q_opt / Cp) / tau * dt
        P_next = 0.5 * (T_history[-1] - T_min)
        T_history.append(T_next)
        P_history.append(P_next)
        Q_history.append(Q_opt)
        opti.set_initial(Q, np.tile(Q_opt, (1, N)))
        opti.set_initial(T, np.tile(T_next, (1, N+1)))
    except:
        print("Solver failed at step", t)
        break

# Plot results
time = np.arange(T_sim)
plt.figure(figsize=(12, 6))
plt.subplot(311)
plt.plot(time, load_demand[:T_sim], 'r--', label='Demand')
plt.plot(time, P_history, 'b-', label='Power Output')
plt.ylabel("Power [MW]")
plt.legend()

plt.subplot(312)
plt.plot(time, T_history[:-1], 'g-', label='Turbine Temp')
plt.ylabel("Temp [°C]")
plt.legend()

plt.subplot(313)
plt.plot(time, Q_history, 'k-', label='Fuel Input')
plt.xlabel("Time [s]")
plt.ylabel("Heat Input")
plt.legend()

plt.tight_layout()
plt.show()
