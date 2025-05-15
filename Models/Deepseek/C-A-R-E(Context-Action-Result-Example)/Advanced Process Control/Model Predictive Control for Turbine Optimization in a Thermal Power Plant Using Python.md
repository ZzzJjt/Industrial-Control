import numpy as np
import matplotlib.pyplot as plt
from casadi import *

# Define the problem dimensions
N = 20  # Number of control intervals
dt = 1.0  # Time step (seconds)
nx = 3  # Number of states (power output, steam flow, fuel input)
nu = 1  # Number of controls (fuel input adjustment)

# State and control bounds
x_min = [0, 0, 0]  # Power output, Steam flow, Fuel input
x_max = [500, 100, 10]  # Power output, Steam flow, Fuel input
u_min = [-1]  # Max negative adjustment
u_max = [1]  # Max positive adjustment

# Initial and final states
x0 = [100, 20, 2]  # Initial power output, steam flow, fuel input
xf = [400, 80, 6]  # Desired power output, steam flow, fuel input

# External disturbance: load demand profile
def load_demand(t):
    if t < 30:
        return 100
    elif t < 60:
        return 300
    else:
        return 400

# Define the symbolic variables
x = MX.sym('x', nx)
u = MX.sym('u', nu)

# Dynamics model: simple first-order linear approximation
A = np.array([
    [0.95, 0.05, 0],
    [0, 0.98, 0.02],
    [0, 0, 0.99]
])
B = np.array([
    [0.1],
    [0.05],
    [0.01]
])

f = A @ x + B @ u

# Create an integrator
F = integrator('F', 'rk4', {'x': x, 'p': u, 'ode': f}, {'tf': dt})

# Cost function: quadratic cost on deviation from goal and control effort
Q = diag([1, 1, 1])  # State weight matrix
R = diag([0.1])  # Control weight matrix

# Initialize optimization problem
opti = Opti()

# Decision variables
X = opti.variable(nx, N+1)  # States over prediction horizon
U = opti.variable(nu, N)    # Controls over prediction horizon

# Initial condition
opti.subject_to(X[:, 0] == x0)

# Constraints
for k in range(N):
    X_next = F(x0=X[:, k], p=U[:, k])['xf']
    opti.subject_to(X[:, k+1] == X_next)
    opti.subject_to(opti.bounded(u_min, U[:, k], u_max))
    opti.subject_to(opti.bounded(x_min, X[:, k], x_max))

# Objective: minimize cost function
obj = 0
for k in range(N):
    obj += mtimes((X[:, k] - xf).T, Q, (X[:, k] - xf)) + \
           mtimes(U[:, k].T, R, U[:, k])
opti.minimize(obj)

# Solve the optimization problem
opti.solver('ipopt')

# PID Controller parameters
Kp = 1.0
Ki = 0.1
Kd = 0.01
integral = 0
last_error = 0

# Simulation loop
X_sim_mpc = np.zeros((nx, N*10))
U_sim_mpc = np.zeros((nu, N*10))
X_sim_pid = np.zeros((nx, N*10))
U_sim_pid = np.zeros((nu, N*10))

X_sim_mpc[:, 0] = x0
X_sim_pid[:, 0] = x0

time_points = np.arange(0, N*10*dt, dt)

for k in range(N*10):
    t = time_points[k]
    demand = load_demand(t)
    
    # MPC Control
    sol = opti.solve()
    X_opt = sol.value(X)
    U_opt = sol.value(U)
    
    X_sim_mpc[:, k] = X_opt[:, 0]
    U_sim_mpc[:, k] = U_opt[:, 0]
    
    # Shift decision variables for next iteration
    opti.set_initial(X[:, :-1], X_opt[:, 1:])
    opti.set_initial(X[:, -1], X_opt[:, -1])
    opti.set_initial(U[:, :-1], U_opt[:, 1:])
    opti.set_initial(U[:, -1], U_opt[:, -1])
    
    # PID Control
    error = demand - X_sim_pid[0, k]
    integral += error * dt
    derivative = (error - last_error) / dt
    pid_output = Kp * error + Ki * integral + Kd * derivative
    last_error = error
    
    U_sim_pid[0, k] = max(min(pid_output, u_max[0]), u_min[0])
    
    # Simulate system using PID control action
    X_next_pid = F(x0=X_sim_pid[:, k], p=U_sim_pid[:, k])['xf']
    X_sim_pid[:, k+1] = X_next_pid
    
    # Update plot
    if k % 10 == 0:
        plt.clf()
        plt.plot(time_points[:k+1], X_sim_mpc[0, :k+1], label='MPC Power Output', color='blue')
        plt.plot(time_points[:k+1], X_sim_pid[0, :k+1], label='PID Power Output', color='orange')
        plt.axhline(demand, color='green', linestyle='--', label='Load Demand')
        plt.xlabel('Time (s)')
        plt.ylabel('Power Output (MW)')
        plt.title('Turbine Power Output Comparison')
        plt.legend()
        plt.pause(0.1)

plt.show()



