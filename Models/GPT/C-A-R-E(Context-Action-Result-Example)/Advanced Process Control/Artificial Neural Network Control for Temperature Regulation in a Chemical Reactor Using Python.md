import numpy as np
import matplotlib.pyplot as plt

# Reactor temperature model: first-order + disturbance
def reactor_temperature(T, u, T_feed, k=0.1, tau=10):
    dTdt = (-T + k*u + 0.5*T_feed) / tau
    return T + dTdt

# Generate synthetic data: u (control), T_feed (disturbance), T (output)
data = []
T = 25  # Initial reactor temperature
for _ in range(1000):
    u = np.random.uniform(0, 100)
    T_feed = np.random.uniform(20, 60)
    T_next = reactor_temperature(T, u, T_feed)
    data.append([u, T_feed, T, T_next])
    T = T_next

data = np.array(data)
X = data[:, :3]  # [u, T_feed, T]
y = data[:, 3]   # T_next

from sklearn.neural_network import MLPRegressor

ann = MLPRegressor(hidden_layer_sizes=(16, 16), max_iter=1000)
ann.fit(X, y)

# PID controller setup
def pid_controller(e, integral, prev_error, Kp=2.0, Ki=0.1, Kd=0.5, dt=1.0):
    integral += e * dt
    derivative = (e - prev_error) / dt
    return Kp * e + Ki * integral + Kd * derivative, integral, e

# Initialize
T_pid = 25
T_ann = 25
T_feed = 30
T_set = 50
u_pid, u_ann = 50, 50
integral, prev_error = 0, 0

T_pid_hist, T_ann_hist = [], []

for t in range(100):
    if t == 50:
        T_feed = 60  # Disturbance at t=50

    # PID
    e = T_set - T_pid
    u_pid, integral, prev_error = pid_controller(e, integral, prev_error)
    T_pid = reactor_temperature(T_pid, u_pid, T_feed)

    # ANN
    x_input = np.array([[u_ann, T_feed, T_ann]])
    T_pred = ann.predict(x_input)[0]
    error = T_set - T_pred
    u_ann += 0.2 * error  # simple correction logic
    T_ann = reactor_temperature(T_ann, u_ann, T_feed)

    T_pid_hist.append(T_pid)
    T_ann_hist.append(T_ann)

  plt.plot(T_pid_hist, label='PID')
plt.plot(T_ann_hist, label='ANN')
plt.axvline(50, color='r', linestyle='--', label='Disturbance')
plt.xlabel('Time step')
plt.ylabel('Reactor Temperature')
plt.title('ANN vs PID Control')
plt.legend()
plt.grid(True)
plt.show()
