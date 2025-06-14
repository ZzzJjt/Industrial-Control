import casadi as ca
import numpy as np
import matplotlib.pyplot as plt

# Constants
T_s = 60  # Sampling time [s]
N = 10    # Prediction horizon
dt = T_s / N  # Time step for discretization

# System parameters
C_t = 1000  # Heat capacity [J/K]
R_t = 10    # Thermal resistance [K/W]
C_p = 1000  # Pressure vessel heat capacity [J/bar]
R_p = 1     # Pressure vessel thermal resistance [bar/W]

# Initial conditions
x0 = np.array([200, 20])  # Initial temperature [°C], initial pressure [bar]

# State and input bounds
lb_x = np.array([200, 20])  # Lower bounds on state (temperature, pressure)
ub_x = np.array([520, 100]) # Upper bounds on state (temperature, pressure)

lb_u = np.array([0, 0])     # Lower bounds on input (heater power, valve opening)
ub_u = np.array([1000, 1])  # Upper bounds on input (heater power, valve opening)

# Reference setpoints
ref_temperature = 500       # Desired temperature [°C]
ref_pressure = 80           # Desired pressure [bar]

# Define the system model (ODEs)
def steam_generator_dynamics(x, u):
    T, P = x
    Q_h, v = u
    
    dT_dt = (Q_h - (T - ref_temperature) / R_t) / C_t
    dP_dt = (v * (ref_pressure - P) / R_p) / C_p
    
    return [dT_dt, dP_dt]

# Discretize the system using forward Euler method
def discrete_system_model(x_k, u_k):
    k1 = steam_generator_dynamics(x_k, u_k)
    x_next = x_k + dt * np.array(k1)
    return x_next

# NMPC setup
opti = ca.Opti()

# Decision variables
X = opti.variable(N+1, 2)  # States (temperature, pressure)
U = opti.variable(N, 2)    # Inputs (heater power, valve opening)

# Objective function
objective = 0
for k in range(N):
    objective += (X[k, 0] - ref_temperature)**2 + (X[k, 1] - ref_pressure)**2

# Initial condition
opti.subject_to(X[0, :] == x0)

# Dynamics constraints
for k in range(N):
    X_next = discrete_system_model(X[k, :], U[k, :])
    opti.subject_to(X[k+1, :] == X_next)

# Input constraints
for k in range(N):
    opti.subject_to(opti.bounded(lb_u, U[k, :], ub_u))

# State constraints
for k in range(N+1):
    opti.subject_to(opti.bounded(lb_x, X[k, :], ub_x))

# Solver options
opti.solver('ipopt', {'print_level': 0})

# Simulation setup
num_steps = 100  # Total number of simulation steps
x_log = np.zeros((num_steps+1, 2))
u_log = np.zeros((num_steps, 2))
x_current = x0.copy()

# Run NMPC
for t in range(num_steps):
    # Set current state as initial guess
    opti.set_initial(X[:, :], np.vstack((np.tile(x_current, (N+1, 1)), np.zeros((N, 2)))))
    
    # Solve the optimization problem
    sol = opti.solve()
    
    # Get optimal control action
    u_opt = sol.value(U)[0, :]
    
    # Store results
    x_log[t, :] = x_current
    u_log[t, :] = u_opt
    
    # Apply control action to the system
    x_current = discrete_system_model(x_current, u_opt)
    
    # Print progress
    print(f"Step {t+1}/{num_steps}: T={x_current[0]:.2f}°C, P={x_current[1]:.2f}bar")

# Log final state
x_log[-1, :] = x_current

# Plot results
plt.figure(figsize=(12, 6))

plt.subplot(2, 1, 1)
plt.plot(np.arange(num_steps+1), x_log[:, 0], label='Temperature [°C]')
plt.axhline(y=ref_temperature, color='r', linestyle='--', label='Reference')
plt.xlabel('Time Steps')
plt.ylabel('Temperature [°C]')
plt.legend()
plt.title('Temperature Response')

plt.subplot(2, 1, 2)
plt.plot(np.arange(num_steps+1), x_log[:, 1], label='Pressure [bar]')
plt.axhline(y=ref_pressure, color='g', linestyle='--', label='Reference')
plt.xlabel('Time Steps')
plt.ylabel('Pressure [bar]')
plt.legend()
plt.title('Pressure Response')

plt.tight_layout()
plt.show()



