import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.preprocessing import MinMaxScaler
from sklearn.neural_network import MLPRegressor
from scipy.integrate import odeint
import control

# Set random seed for reproducibility
np.random.seed(42)

# Step 1: Simulate historical reactor data (nonlinear and time-varying)
def reactor_dynamics(T, t, Q, T_cool, k0=1e5, E=4e4, R=8.314, deltaH=5e4, rho=1000, Cp=1):
    # Nonlinear reactor model: dT/dt = f(T, Q, T_cool)
    # T: reactor temperature (K), Q: heat input (W), T_cool: coolant temp (K)
    k = k0 * np.exp(-E / (R * T))  # Reaction rate (time-varying)
    dTdt = (Q - deltaH * k * rho - 0.1 * (T - T_cool)) / (rho * Cp)
    return dTdt

# Generate historical data
t = np.linspace(0, 1000, 10000)
Q = 5000 + 2000 * np.sin(0.01 * t) + np.random.normal(0, 200, t.size)  # Heat input
T_cool = 300 + 10 * np.sin(0.005 * t)  # Coolant temperature
T0 = 350  # Initial temperature
T = odeint(reactor_dynamics, T0, t, args=(Q, T_cool)).flatten()

# Create DataFrame
data = pd.DataFrame({'time': t, 'T': T, 'Q': Q, 'T_cool': T_cool})
data['T_next'] = data['T'].shift(-1)  # Target: next time step temperature
data = data.dropna()

# Step 2: Data preprocessing
scaler = MinMaxScaler()
X = scaler.fit_transform(data[['T', 'Q', 'T_cool']])  # Inputs
y = scaler.fit_transform(data[['T_next']]).flatten()  # Output

# Split data
train_size = int(0.8 * len(X))
X_train, X_test = X[:train_size], X[train_size:]
y_train, y_test = y[:train_size], y[train_size:]

# Step 3: Train ANN model
ann = MLPRegressor(hidden_layer_sizes=(64, 32), activation='relu', solver='adam',
                  max_iter=1000, random_state=42, verbose=False)
ann.fit(X_train, y_train)

# Step 4: Define ANN controller
def ann_controller(T_current, T_setpoint, T_cool, scaler, model, Q_prev):
    # Predict optimal Q to reach setpoint
    X = scaler.transform([[T_current, Q_prev, T_cool]])
    T_pred = model.predict(X)[0]
    T_pred = scaler.inverse_transform([[T_pred]])[0, 0]
    
    # Simple control logic: adjust Q based on error
    error = T_setpoint - T_current
    Q = Q_prev + 100 * error * (1 + 0.1 * (T_pred - T_current))
    return np.clip(Q, 0, 10000)  # Limit heat input

# Step 5: Define PID controller
def pid_controller(T_current, T_setpoint, dt, pid_params, integral=0, prev_error=0):
    Kp, Ki, Kd = pid_params
    error = T_setpoint - T_current
    integral += error * dt
    derivative = (error - prev_error) / dt
    Q = Kp * error + Ki * integral + Kd * derivative
    return np.clip(Q, 0, 10000), integral, error

# Step 6: Simulate control performance
def simulate_control(T_setpoint, controller_type, pid_params=None):
    T = 350  # Initial temperature
    Q = 5000  # Initial heat input
    T_cool = 300
    dt = 0.1
    t_sim = np.arange(0, 200, dt)
    T_history = [T]
    Q_history = [Q]
    integral, prev_error = 0, 0
    
    for _ in range(1, len(t_sim)):
        if controller_type == 'ANN':
            Q = ann_controller(T, T_setpoint, T_cool, scaler, ann, Q)
        else:  # PID
            Q, integral, prev_error = pid_controller(T, T_setpoint, dt, pid_params, integral, prev_error)
        
        # Update reactor state
        T = odeint(reactor_dynamics, T, [0, dt], args=(Q, T_cool))[-1, 0]
        T_history.append(T)
        Q_history.append(Q)
        T_cool = 300 + 10 * np.sin(0.05 * t_sim[_])  # Time-varying coolant
    
    return t_sim, T_history, Q_history

# Step 7: Run simulations for different setpoints
setpoints = [360, 370, 380]
pid_params = (100, 0.5, 10)  # Tuned PID parameters
results = {}

for sp in setpoints:
    # ANN control
    t_sim, T_ann, Q_ann = simulate_control(sp, 'ANN')
    # PID control
    t_sim, T_pid, Q_pid = simulate_control(sp, 'PID', pid_params)
    results[sp] = {'ANN': (T_ann, Q_ann), 'PID': (T_pid, Q_pid), 'time': t_sim}

# Step 8: Evaluate and visualize results
plt.figure(figsize=(12, 8))
for i, sp in enumerate(setpoints, 1):
    plt.subplot(3, 1, i)
    plt.plot(results[sp]['time'], results[sp]['ANN'][0], label='ANN Control')
    plt.plot(results[sp]['time'], results[sp]['PID'][0], label='PID Control')
    plt.axhline(sp, color='r', linestyle='--', label='Setpoint')
    plt.title(f'Temperature Control at Setpoint {sp} K')
    plt.xlabel('Time (s)')
    plt.ylabel('Temperature (K)')
    plt.legend()
    plt.grid(True)

plt.tight_layout()
plt.savefig('control_comparison.png')

# Step 9: Calculate performance metrics
def calculate_metrics(T_history, T_setpoint):
    error = np.abs(np.array(T_history) - T_setpoint)
    mse = np.mean(error**2)
    settling_time = next((i * 0.1 for i, e in enumerate(error[100:]) if e < 0.01 * T_setpoint), 200)
    return mse, settling_time

print("Performance Metrics:")
for sp in setpoints:
    mse_ann, st_ann = calculate_metrics(results[sp]['ANN'][0], sp)
    mse_pid, st_pid = calculate_metrics(results[sp]['PID'][0], sp)
    print(f"\nSetpoint {sp} K:")
    print(f"ANN - MSE: {mse_ann:.2f}, Settling Time: {st_ann:.1f} s")
    print(f"PID - MSE: {mse_pid:.2f}, Settling Time: {st_pid:.1f} s")
