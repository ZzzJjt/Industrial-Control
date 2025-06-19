import numpy as np
from scipy.integrate import solve_ivp
from casadi import *

# Physical constants and constraints
T_max = 560    # °C
T_min = 400    # °C
p_max = 170    # bar
p_min = 80     # bar
P_max = 600    # MW
P_min = 100    # MW
dT_max = 2.0   # °C/min
dp_max = 3.0   # bar/min
dP_max = 5.0   # MW/min

# First-order approximation of turbine dynamics
def turbine_dynamics(t, x, u):
    T, p, P = x
    m_dot, _ = u  # Ramp rate is handled via MPC constraints

    dTdt = 0.05 * (m_dot - x[0])  # Simplified heat transfer dynamics
    dpdt = 0.08 * (m_dot - x[1])  # Pressure buildup from steam flow
    dPdt = 0.1 * m_dot            # Linear relation between flow and power generation

    return [dTdt, dpdt, dPdt]

opti = Opti()

# Decision variables
X = opti.variable(N+1, 3)  # States: [Temperature, Pressure, Power]
U = opti.variable(N, 2)    # Inputs: [Mass Flow, Ramp Rate]

# Parameters
P_setpoint = opti.parameter()         # Target power setpoint
P_disturbance = opti.parameter(2, N)  # Disturbances: ambient temp, pressure loss

# Initial state
X0 = opti.parameter(3)
opti.subject_to(X[0, :] == X0)

# Dynamics constraints
for k in range(N):
    dx = turbine_dynamics(0, X[k, :], U[k, :])
    opti.subject_to(X[k+1, :] == X[k, :] + dx*dt)

# State constraints
for k in range(N+1):
    opti.subject_to(opti.bounded(T_min, X[k, 0], T_max))
    opti.subject_to(opti.bounded(p_min, X[k, 1], p_max))
    opti.subject_to(opti.bounded(P_min, X[k, 2], P_max))

# Input constraints
for k in range(N):
    opti.subject_to(opti.bounded(0, U[k, 0], 1000))       # Mass flow limits
    opti.subject_to(opti.bounded(-dP_max, U[k, 1], dP_max))  # Ramp rate limits

# Objective function
cost = 0
for k in range(N):
    cost += (X[k, 2] - P_setpoint)**2 + 0.1*U[k, 0]**2 + 0.05*(U[k, 1])**2
opti.minimize(cost)

# Set solver
opti.solver('ipopt')

# Efficiency metric: Power per unit mass flow
for k in range(N):
    efficiency = X[k, 2] / U[k, 0]
    opti.subject_to(efficiency >= 0.5)  # Example constraint

# Safety: Avoid rapid changes that could cause thermal stress
for k in range(N):
    if k > 0:
        dT = abs(X[k, 0] - X[k-1, 0])
        dp = abs(X[k, 1] - X[k-1, 1])
        opti.subject_to(dT <= dT_max * dt)
        opti.subject_to(dp <= dp_max * dt)

# Efficiency metric: Power per unit mass flow
for k in range(N):
    efficiency = X[k, 2] / U[k, 0]
    opti.subject_to(efficiency >= 0.5)  # Example constraint

# Safety: Avoid rapid changes that could cause thermal stress
for k in range(N):
    if k > 0:
        dT = abs(X[k, 0] - X[k-1, 0])
        dp = abs(X[k, 1] - X[k-1, 1])
        opti.subject_to(dT <= dT_max * dt)
        opti.subject_to(dp <= dp_max * dt)

# Simulation setup
T_sim = 30  # minutes
N_sim = int(T_sim / dt)

state = np.array([500, 150, 400])  # Initial conditions
history = []

load_profile = np.concatenate((np.ones(100)*400,
                               np.linspace(400, 550, 50),
                               np.ones(100)*550,
                               np.linspace(550, 300, 50)))

for t in range(N_sim):
    if t < len(load_profile):
        current_setpoint = load_profile[t]
    else:
        current_setpoint = 300

    # Update disturbance parameters
    ambient_temp = 25 + 10*np.sin(2*np.pi*t*dt / 60)
    pressure_loss = 2*np.random.rand()

    opti.set_value(P_setpoint, current_setpoint)
    opti.set_value(P_disturbance, [ambient_temp, pressure_loss])
    opti.set_value(X0, state)

    try:
        sol = opti.solve()
        u_opt = sol.value(U)[0]
    except:
        u_opt = [0, 0]

    # Apply control input
    dx = turbine_dynamics(0, state, u_opt)
    state += dx * dt

    history.append(state.copy())

history = np.array(history)

# Final power vs setpoint
final_power = history[-1, 2]
tracking_error = np.mean(np.abs(history[:, 2] - load_profile[:len(history)]))

# Efficiency
avg_efficiency = np.mean(history[:, 2] / history[:, 0])

print(f"Final Power: {final_power:.2f} MW")
print(f"Avg Tracking Error: {tracking_error:.2f} MW")
print(f"Avg Efficiency: {avg_efficiency:.2f} MW/°C")
print(f"Constraint Violations: {np.sum((history[:, 0] > T_max) | (history[:, 0] < T_min))}")
