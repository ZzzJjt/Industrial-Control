import casadi as ca
import numpy as np
import matplotlib.pyplot as plt

# Process constraints
T_ambient = 25.0
P_ambient = 1.0
T_max = 520.0
P_min = 30.0

# States and input
T = ca.MX.sym('T')
P = ca.MX.sym('P')
u = ca.MX.sym('u')

# Dynamics
dTdt = -0.01*(T - T_ambient) + 0.05*u
dPdt = -0.02*(P - P_ambient) + 0.01*u*T/100.0
x = ca.vertcat(T, P)
xdot = ca.vertcat(dTdt, dPdt)
f = ca.Function('f', [x, u], [xdot])

# NMPC setup
N = 20
dt = 1.0
x0 = np.array([100.0, 5.0])
T_target = 500.0
P_target = 35.0

X = ca.MX.sym('X', 2, N+1)
U = ca.MX.sym('U', 1, N)

obj = 0
g = []

for k in range(N):
    x_k = X[:, k]
    u_k = U[:, k]
    x_next = X[:, k+1]
    x_next_pred = x_k + dt * f(x_k, u_k)

    obj += ca.sumsqr(x_k - ca.vertcat(T_target, P_target)) + 0.01*ca.sumsqr(u_k)
    g.append(x_next - x_next_pred)

obj += ca.sumsqr(X[:, -1] - ca.vertcat(T_target, P_target))
g = ca.vertcat(*g)

vars = ca.vertcat(ca.reshape(X, -1, 1), ca.reshape(U, -1, 1))
nlp = {'x': vars, 'f': obj, 'g': g}

solver = ca.nlpsol('solver', 'ipopt', nlp)

x_guess = np.tile(x0.reshape(-1, 1), (1, N+1))
u_guess = np.zeros((1, N))
vars_init = np.concatenate([x_guess.flatten(), u_guess.flatten()])
lbg = np.zeros(g.shape)
ubg = np.zeros(g.shape)

lbx = []
ubx = []
for _ in range(N+1):
    lbx.extend([0.0, P_min])
    ubx.extend([T_max, 100.0])
for _ in range(N):
    lbx.append(0.0)
    ubx.append(100.0)

sol = solver(x0=vars_init, lbg=lbg, ubg=ubg, lbx=lbx, ubx=ubx)
sol_vals = sol['x'].full().flatten()
X_vals = sol_vals[:2*(N+1)].reshape((2, N+1))
U_vals = sol_vals[2*(N+1):].reshape((1, N))

plt.plot(range(N+1), X_vals[0], label="Temperature (°C)")
plt.plot(range(N+1), X_vals[1], label="Pressure (bar)")
plt.plot(range(N), U_vals[0], label="Heating rate (u)", linestyle='--')
plt.axhline(y=T_max, color='r', linestyle=':', label='T_max')
plt.axhline(y=P_min, color='g', linestyle=':', label='P_min')
plt.legend()
plt.xlabel('Time step')
plt.title('NMPC: Steam Generator Startup')
plt.grid(True)
plt.tight_layout()
plt.show()
