import numpy as np
import matplotlib.pyplot as plt
from sklearn.neural_network import MLPRegressor

# Simulate nonlinear reactor temperature dynamics
def reactor_dynamics(temp, control_input, time):
    # Nonlinear and time-varying behavior
    alpha = 0.01 * (1 + 0.5 * np.sin(0.1 * time))  # time-varying gain
    disturbance = 2.0 * np.sin(0.05 * time)       # external disturbance
    return temp + alpha * (control_input - temp**2) + disturbance

# Generate training data for ANN
def generate_training_data():
    X, y = [], []
    for t in range(2000):
        temp = np.random.uniform(40, 80)
        control_input = np.random.uniform(0, 100)
        next_temp = reactor_dynamics(temp, control_input, t)
        X.append([temp, control_input, t])
        y.append(next_temp)
    return np.array(X), np.array(y)

# Train ANN
X_train, y_train = generate_training_data()
ann = MLPRegressor(hidden_layer_sizes=(20, 20), max_iter=1000)
ann.fit(X_train, y_train)

# Compare ANN vs PID controller
def simulate_controller(use_ann=True):
    temps = [50.0]
    setpoint = 70.0
    integral = 0
    prev_error = 0
    kp, ki, kd = 0.5, 0.01, 0.05

    for t in range(1, 300):
        current_temp = temps[-1]
        error = setpoint - current_temp
        integral += error
        derivative = error - prev_error
        prev_error = error

        if use_ann:
            control_input = ann.predict([[current_temp, error, t]])[0]
        else:
            control_input = kp * error + ki * integral + kd * derivative

        next_temp = reactor_dynamics(current_temp, control_input, t)
        temps.append(next_temp)

    return temps

# Plot results
time = np.arange(301)
pid_result = simulate_controller(use_ann=False)
ann_result = simulate_controller(use_ann=True)

plt.plot(time, pid_result, label="PID Controller")
plt.plot(time, ann_result, label="ANN Controller")
plt.axhline(y=70, color='gray', linestyle='--', label="Setpoint")
plt.xlabel("Time")
plt.ylabel("Reactor Temperature")
plt.title("ANN vs PID Control of Nonlinear Reactor")
plt.legend()
plt.grid()
plt.show()
