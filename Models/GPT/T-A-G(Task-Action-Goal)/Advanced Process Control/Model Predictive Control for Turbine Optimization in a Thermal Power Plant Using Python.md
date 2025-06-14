import numpy as np
import matplotlib.pyplot as plt
from scipy.linalg import solve_discrete_are

# Define a simplified thermal power plant turbine model
# x[k+1] = Ax[k] + Bu[k] + Ed[k] (E accounts for external disturbances)
A = np.array([[0.95]])         # Turbine speed retention
B = np.array([[0.1]])          # Control effect (valve position)
E = np.array([[0.05]])         # Disturbance impact (load demand)
C = np.array([[1.0]])          # Measured output

# Discretization step
Ts = 1.0  # 1 second sampling

# Define MPC parameters
Q = np.array([[10]])   # Output weight
R = np.array([[1]])    # Control weight
N = 10                 # Prediction horizon

# Solve discrete-time Algebraic Riccati Equation for LQR gain (MPC simplification)
P = solve_discrete_are(A, B, Q, R)
K = -np.linalg.inv(R + B.T @ P @ B) @ B.T @ P @ A

# Simulation setup
sim_time = 60
x = np.array([[0.0]])
u = []
y = []
d = 0.0  # Initial disturbance (can vary during sim)

# Reference trajectory: ramping turbine speed
r = np.linspace(0, 1.0, sim_time)

for k in range(sim_time):
    # Simulate load disturbance
    if k == 30:
        d = 0.5  # sudden load change

    # Compute control input using LQR/MPC formulation
    error = x - r[k]
    u_k = float(K @ error)
    u_k = np.clip(u_k, -1.0, 1.0)  # Actuator constraint (valve position limits)

    # System update
    x = A @ x + B * u_k + E * d
    y.append(float(C @ x))
    u.append(u_k)

# Plot turbine response
plt.figure(figsize=(10, 6))
plt.plot(r, 'k--', label='Reference Speed')
plt.plot(y, label='Turbine Speed')
plt.plot(u, label='Valve Position (Control Input)')
plt.xlabel('Time (s)')
plt.ylabel('Turbine Response')
plt.title('MPC-Based Turbine Speed Control')
plt.grid(True)
plt.legend()
plt.show()
