import numpy as np
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
import matplotlib.pyplot as plt
from scipy.integrate import odeint

# 1. Generate Synthetic Historical Data
def generate_reactor_data(n_samples=10000, dt=0.1):
    """
    Generate synthetic data for reactor temperature, control input, and disturbance.
    Reactor model: dT/dt = -a*T + b*u + c*d + nonlinearity
    """
    np.random.seed(42)
    T = np.zeros(n_samples)  # Temperature (°C)
    u = np.zeros(n_samples)  # Control input (coolant flow, normalized 0–1)
    d = np.zeros(n_samples)  # Disturbance (feed variation, normalized 0–1)
    
    # Initial conditions
    T[0] = 50.0
    u[0] = 0.5
    d[0] = 0.2
    
    # Parameters
    a = 0.1  # Thermal decay
    b = 20.0  # Control effect
    c = 10.0  # Disturbance effect
    
    for i in range(1, n_samples):
        # Random control and disturbance
        u[i] = np.clip(u[i-1] + np.random.normal(0, 0.05), 0, 1)
        d[i] = np.clip(d[i-1] + np.random.normal(0, 0.02), 0, 1)
        
        # Nonlinear reactor dynamics
        dTdt = -a * T[i-1] + b * u[i-1] + c * d[i-1] + 0.05 * T[i-1]**2 / (T[i-1] + 100)
        T[i] = T[i-1] + dt * dTdt + np.random.normal(0, 0.1)  # Add noise
    
    return T, u, d

# Generate data
n_samples = 10000
dt = 0.1
T, u, d = generate_reactor_data(n_samples, dt)

# Prepare training data for ANN
def prepare_data(T, u, d, lookback=5):
    X = []
    y = []
    for i in range(lookback, len(T)):
        X.append(np.concatenate([
            T[i-lookback:i],  # Past temperatures
            u[i-lookback:i],  # Past control inputs
            d[i-lookback:i]   # Past disturbances
        ]))
        y.append(T[i])  # Predict next temperature
    return np.array(X), np.array(y)

lookback = 5
X, y = prepare_data(T, u, d, lookback)

# Split data
train_size = int(0.8 * len(X))
X_train, X_test = X[:train_size], X[train_size:]
y_train, y_test = y[:train_size], y[train_size:]

# 2. Build and Train ANN Model
def build_ann_model(input_dim):
    model = Sequential([
        Dense(64, activation='relu', input_shape=(input_dim,)),
        Dense(32, activation='relu'),
        Dense(16, activation='relu'),
        Dense(1)  # Predict temperature
    ])
    model.compile(optimizer='adam', loss='mse')
    return model

# Train ANN
input_dim = lookback * 3  # T, u, d for each lookback step
ann_model = build_ann_model(input_dim)
history = ann_model.fit(X_train, y_train, epochs=20, batch_size=32, 
                       validation_data=(X_test, y_test), verbose=0)

# 3. Simulate Reactor Dynamics
def reactor_model(state, t, u, d, a=0.1, b=20.0, c=10.0):
    T = state[0]
    dTdt = -a * T + b * u + c * d + 0.05 * T**2 / (T + 100)
    return [dTdt]

def simulate_reactor(T0, u_traj, d_traj, dt, t_span):
    T = [T0]
    state = [T0]
    for i in range(len(t_span)-1):
        t = [t_span[i], t_span[i+1]]
        sol = odeint(reactor_model, state, t, args=(u_traj[i], d_traj[i]))
        state = sol[-1]
        T.append(state[0])
    return np.array(T)

# ANN-based Controller
def ann_controller(T_history, u_history, d_history, setpoint, ann_model, lookback):
    """
    Predict optimal control input using ANN model.
    """
    if len(T_history) < lookback:
        return 0.5  # Default control
    input_data = np.concatenate([
        T_history[-lookback:],
        u_history[-lookback:],
        d_history[-lookback:]
    ]).reshape(1, -1)
    
    # Optimize control input (simple grid search for demonstration)
    u_candidates = np.linspace(0, 1, 20)
    errors = []
    for u in u_candidates:
        input_data[0, lookback-1] = u  # Replace latest u
        T_pred = ann_model.predict(input_data, verbose=0)[0][0]
        errors.append(abs(T_pred - setpoint))
    return u_candidates[np.argmin(errors)]

# PID Controller
class PIDController:
    def __init__(self, Kp=0.5, Ki=0.1, Kd=0.05, dt=0.1):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.dt = dt
        self.integral = 0
        self.prev_error = 0
    
    def compute(self, setpoint, T):
        error = setpoint - T
        self.integral += error * self.dt
        derivative = (error - self.prev_error) / self.dt
        output = self.Kp * error + self.Ki * self.integral + self.Kd * derivative
        self.prev_error = error
        return np.clip(output, 0, 1)

# Simulation Setup
t_span = np.arange(0, 100, dt)
setpoint = 60.0  # Target temperature (°C)
T0 = 50.0  # Initial temperature

# Disturbance: Step and ramp
d_traj = np.ones(len(t_span)) * 0.2
d_traj[200:400] += 0.3  # Step disturbance
d_traj[600:800] += np.linspace(0, 0.4, 200)  # Ramp disturbance

# ANN Controller Simulation
T_ann = [T0]
u_ann = [0.5]
d_history = [0.2]
T_history = [T0] * lookback
u_history = [0.5] * lookback
d_history = [0.2] * lookback

for i in range(1, len(t_span)):
    u = ann_controller(T_history, u_history, d_history, setpoint, ann_model, lookback)
    u_ann.append(u)
    T_next = simulate_reactor(T_ann[-1], [u], [d_traj[i]], dt, [t_span[i], t_span[i+1]])[-1]
    T_ann.append(T_next)
    T_history = T_history[1:] + [T_next]
    u_history = u_history[1:] + [u]
    d_history = d_history[1:] + [d_traj[i]]

# PID Controller Simulation
pid = PIDController(Kp=0.5, Ki=0.1, Kd=0.05, dt=dt)
T_pid = [T0]
u_pid = [0.5]

for i in range(1, len(t_span)):
    u = pid.compute(setpoint, T_pid[-1])
    u_pid.append(u)
    T_next = simulate_reactor(T_pid[-1], [u], [d_traj[i]], dt, [t_span[i], t_span[i+1]])[-1]
    T_pid.append(T_next)

# 4. Performance Comparison
def calculate_metrics(T, setpoint, dt):
    error = setpoint - T
    iae = np.sum(np.abs(error)) * dt  # Integral Absolute Error
    overshoot = max(0, max(T - setpoint))  # Maximum overshoot
    settling_time = 0
    for i, e in enumerate(error):
        if abs(e) > 0.02 * setpoint and i * dt > 10:
            settling_time = i * dt
    return iae, overshoot, settling_time

iae_ann, overshoot_ann, settling_ann = calculate_metrics(np.array(T_ann), setpoint, dt)
iae_pid, overshoot_pid, settling_pid = calculate_metrics(np.array(T_pid), setpoint, dt)

print("ANN Controller Metrics:")
print(f"IAE: {iae_ann:.2f}, Overshoot: {overshoot_ann:.2f}°C, Settling Time: {settling_ann:.2f}s")
print("PID Controller Metrics:")
print(f"IAE: {iae_pid:.2f}, Overshoot: {overshoot_pid:.2f}°C, Settling Time: {settling_pid:.2f}s")

# Plot Results
plt.figure(figsize=(12, 8))
plt.subplot(2, 1, 1)
plt.plot(t_span, T_ann, label='ANN Temperature')
plt.plot(t_span, T_pid, label='PID Temperature')
plt.axhline(setpoint, color='r', linestyle='--', label='Setpoint')
plt.title('Temperature Control Comparison')
plt.xlabel('Time (s)')
plt.ylabel('Temperature (°C)')
plt.legend()
plt.grid()

plt.subplot(2, 1, 2)
plt.plot(t_span, u_ann, label='ANN Control Input')
plt.plot(t_span, u_pid, label='PID Control Input')
plt.plot(t_span, d_traj, label='Disturbance', linestyle='--')
plt.title('Control Input and Disturbance')
plt.xlabel('Time (s)')
plt.ylabel('Control/Disturbance')
plt.legend()
plt.grid()

plt.tight_layout()
plt.savefig('reactor_control_comparison.png')
plt.close()
