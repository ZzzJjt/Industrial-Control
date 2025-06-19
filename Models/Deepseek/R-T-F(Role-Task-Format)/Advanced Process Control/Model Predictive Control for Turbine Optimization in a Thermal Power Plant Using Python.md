import numpy as np
import cvxpy as cp
import matplotlib.pyplot as plt

# Constants
Ts = 0.1  # Sampling time [seconds]
Np = 10   # Prediction horizon
Nu = 1    # Number of control inputs (steam flow rate)
Nx = 3    # Number of states (temperature, pressure, energy output)

# Dynamic Model: Simplified Turbine Model
def turbine_model(x, u):
    """
    x: state vector [temperature, pressure, energy_output]
    u: input vector [steam_flow_rate]
    """
    T = x[0]
    P = x[1]
    E = x[2]
    
    # Parameters
    k_T = 0.1  # Temperature gain
    k_P = 0.2  # Pressure gain
    k_E = 0.3  # Energy output gain
    
    dT_dt = k_T * u - 0.05 * T
    dP_dt = k_P * u - 0.01 * P
    dE_dt = k_E * u - 0.02 * E
    
    return np.array([dT_dt, dP_dt, dE_dt])

# Initialize matrices for QP
Q = np.diag([1.0, 1.0, 1.0])  # State weight matrix
R = np.diag([0.1])              # Input weight matrix

# Define operational constraints
u_min = 0.0  # Minimum steam flow rate
u_max = 10.0 # Maximum steam flow rate
x_min = np.array([200, 10, 0]) # Minimum state values [temperature, pressure, energy_output]
x_max = np.array([600, 50, 10])# Maximum state values [temperature, pressure, energy_output]

# Define goal setpoints
goal_temperature = 500  # Desired temperature [°C]
goal_pressure = 30      # Desired pressure [bar]
goal_energy_output = 8  # Desired energy output [MW]

# Define external disturbances (e.g., varying load demand)
def disturbance(t):
    if t < 20:
        return 0.0
    elif t < 40:
        return 2.0
    else:
        return 0.0

# MPC Controller
class MPCController:
    def __init__(self, nx, nu, np, Q, R, Ts, goal_state, x_min, x_max, u_min, u_max):
        self.nx = nx
        self.nu = nu
        self.np = np
        self.Q = Q
        self.R = R
        self.Ts = Ts
        self.goal_state = goal_state
        self.x_min = x_min
        self.x_max = x_max
        self.u_min = u_min
        self.u_max = u_max

    def compute_control(self, x0, disturbance_val):
        # Decision variables
        X = cp.Variable((self.nx, self.np))
        U = cp.Variable((self.nu, self.np-1))

        # Objective function
        objective = 0
        for k in range(self.np):
            objective += cp.quad_form(X[:, k] - self.goal_state, self.Q)
        for k in range(self.np-1):
            objective += cp.quad_form(U[:, k], self.R)

        # Constraints
        constraints = []
        constraints += [X[:, 0] == x0]
        for k in range(self.np-1):
            constraints += [X[:, k+1] == X[:, k] + self.Ts * turbine_model(X[:, k], U[:, k])]
            constraints += [U[:, k] >= self.u_min]
            constraints += [U[:, k] <= self.u_max]
            constraints += [X[:, k] >= self.x_min]
            constraints += [X[:, k] <= self.x_max]
        
        # Add disturbance effect
        constraints += [X[:, 0] == x0 + disturbance_val]

        # Problem setup
        prob = cp.Problem(cp.Minimize(objective), constraints)

        # Solve problem
        prob.solve(solver=cp.OSQP)

        # Return first control action
        if prob.status == cp.OPTIMAL:
            return U[:, 0].value
        else:
            print("Optimization failed!")
            return np.zeros(self.nu)

# Simulation Settings
simulation_steps = 100
x_history = [initial_state]
u_history = []

# Initial state
initial_state = np.array([300, 15, 2])  # Initial [temperature, pressure, energy_output]

# Create MPC controller
mpc_controller = MPCController(Nx, Nu, Np, Q, R, Ts, np.array([goal_temperature, goal_pressure, goal_energy_output]), x_min, x_max, u_min, u_max)

# Simulation Loop
current_state = initial_state.copy()
for t in range(simulation_steps):
    # Compute disturbance
    disturbance_val = disturbance(t * Ts)
    
    # Compute control action
    u_optimal = mpc_controller.compute_control(current_state, disturbance_val)
    
    # Store history
    x_history.append(current_state)
    u_history.append(u_optimal)
    
    # Update state using motion model
    current_state += Ts * turbine_model(current_state, u_optimal)
    
    # Print current state and control action
    print(f"Time: {t*Ts:.2f} s")
    print(f"State: Temp={current_state[0]:.2f} °C, Press={current_state[1]:.2f} bar, Energy={current_state[2]:.2f} MW")
    print(f"Control: Steam Flow Rate={u_optimal[0]:.2f} kg/s")
    print(f"Disturbance: {disturbance_val:.2f}")
    print("----------------------------------------")

# Convert history lists to arrays for plotting
x_history = np.array(x_history)
u_history = np.array(u_history)

# Optional: Plotting
plt.figure(figsize=(12, 8))

# Plot states
plt.subplot(2, 2, 1)
plt.plot(np.arange(simulation_steps + 1) * Ts, x_history[:, 0], label='Temperature')
plt.axhline(y=goal_temperature, color='r', linestyle='--', label='Goal')
plt.xlabel('Time [s]')
plt.ylabel('Temperature [°C]')
plt.title('Temperature Over Time')
plt.legend()
plt.grid(True)

plt.subplot(2, 2, 2)
plt.plot(np.arange(simulation_steps + 1) * Ts, x_history[:, 1], label='Pressure')
plt.axhline(y=goal_pressure, color='r', linestyle='--', label='Goal')
plt.xlabel('Time [s]')
plt.ylabel('Pressure [bar]')
plt.title('Pressure Over Time')
plt.legend()
plt.grid(True)

plt.subplot(2, 2, 3)
plt.plot(np.arange(simulation_steps + 1) * Ts, x_history[:, 2], label='Energy Output')
plt.axhline(y=goal_energy_output, color='r', linestyle='--', label='Goal')
plt.xlabel('Time [s]')
plt.ylabel('Energy Output [MW]')
plt.title('Energy Output Over Time')
plt.legend()
plt.grid(True)

# Plot controls
plt.subplot(2, 2, 4)
plt.plot(np.arange(simulation_steps) * Ts, u_history, label='Steam Flow Rate')
plt.xlabel('Time [s]')
plt.ylabel('Steam Flow Rate [kg/s]')
plt.title('Control Actions Over Time')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.show()
