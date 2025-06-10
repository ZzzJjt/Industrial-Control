import numpy as np
import matplotlib.pyplot as plt
from sklearn.neural_network import MLPRegressor
from scipy.integrate import odeint

# ----------------------------
# 1. Define Nonlinear Reactor Dynamics
# ----------------------------
def reactor_dynamics(T, t, Q, k=0.05, T_env=25, alpha=0.1):
    """Nonlinear and time-varying dynamics of reactor temperature."""
    # k: time-varying heat loss, alpha: nonlinearity factor
    k_eff = k + 0.01 * np.sin(0.1 * t)  # time-varying component
    dTdt = -k_eff * (T - T_env) + alpha * Q**2  # nonlinear term
    return dTdt

# ----------------------------
# 2. Generate Historical Training Data
# ----------------------------
t_hist = np.linspace(0, 50, 500)
Q_hist = np.random.uniform(0, 10, size=len(t_hist))
T_hist = np.zeros_like(t_hist)
T = 25

for i in range(len(t_hist)):
    dt = t_hist[1] - t_hist[0]
    dT = reactor_dynamics(T, t_hist[i], Q_hist[i]) * dt
    T += dT
    T_hist[i] = T

# Create supervised learning data
X_train = np.vstack([Q_hist[:-1], T_hist[:-1]]).T
y_train = T_hist[1:]

# ----------------------------
# 3. Train ANN Model
# ----------------------------
model = MLPRegressor(hidden_layer_sizes=(20, 20), activation='relu', max_iter=1000)
model.fit(X_train, y_train)

# ----------------------------
# 4. Define ANN-Based Controller
# ----------------------------
def ann_controller(T_current, T_setpoint):
    """Simple inverse model controller using brute-force ANN prediction."""
    best_Q = 0
    best_error = float('inf')
    for Q in np.linspace(0, 10, 100):
        T_pred = model.predict([[Q, T_current]])[0]
        error = abs(T_pred - T_setpoint)
        if error < best_error:
            best_error = error
            best_Q = Q
    return best_Q

# ----------------------------
# 5. Define PID Controller
# ----------------------------
class PID:
    def __init__(self, Kp, Ki, Kd):
        self.Kp, self.Ki, self.Kd = Kp, Ki, Kd
        self.integral = 0
        self.prev_error = 0

    def compute(self, error, dt):
        self.integral += error * dt
        derivative = (error - self.prev_error) / dt
        self.prev_error = error
        return self.Kp * error + self.Ki * self.integral + self.Kd * derivative

# ----------------------------
# 6. Simulation: ANN vs PID
# ----------------------------
T_setpoint = 80
t_sim = np.linspace(0, 100, 1000)
T_ann, T_pid = 25, 25
Q_ann_hist, Q_pid_hist = [], []
T_ann_hist, T_pid_hist = [], []

pid = PID(Kp=2.0, Ki=0.2, Kd=0.1)

for i in range(len(t_sim)):
    dt = t_sim[1] - t_sim[0]

    # ANN control
    Q_ann = ann_controller(T_ann, T_setpoint)
    T_ann += reactor_dynamics(T_ann, t_sim[i], Q_ann) * dt

    # PID control
    error = T_setpoint - T_pid
    Q_pid = pid.compute(error, dt)
    Q_pid = np.clip(Q_pid, 0, 10)
    T_pid += reactor_dynamics(T_pid, t_sim[i], Q_pid) * dt

    # Save data
    Q_ann_hist.append(Q_ann)
    Q_pid_hist.append(Q_pid)
    T_ann_hist.append(T_ann)
    T_pid_hist.append(T_pid)

# ----------------------------
# 7. Plot Results
# ----------------------------
plt.figure(figsize=(12, 6))

plt.subplot(2, 1, 1)
plt.plot(t_sim, T_ann_hist, label='ANN Controller')
plt.plot(t_sim, T_pid_hist, label='PID Controller')
plt.axhline(T_setpoint, color='gray', linestyle='--', label='Setpoint')
plt.ylabel('Temperature (°C)')
plt.title('Temperature Response')
plt.legend()

plt.subplot(2, 1, 2)
plt.plot(t_sim, Q_ann_hist, label='Q (ANN)')
plt.plot(t_sim, Q_pid_hist, label='Q (PID)')
plt.xlabel('Time (s)')
plt.ylabel('Control Input (Q)')
plt.title('Control Effort')
plt.legend()

plt.tight_layout()
plt.show()
