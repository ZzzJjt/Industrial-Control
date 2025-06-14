import numpy as np
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
from sklearn.preprocessing import MinMaxScaler
import matplotlib.pyplot as plt

# Mock historical data generation for training
time_steps = 1000
t = np.arange(time_steps)
temp_data = 100 + 10 * np.sin(0.02 * t) + 5 * np.random.randn(time_steps)
control_data = np.clip(0.5 * temp_data + 5 * np.random.randn(time_steps), 0, 100)

# Scaling data
scaler = MinMaxScaler()
inputs = scaler.fit_transform(control_data.reshape(-1, 1))
targets = scaler.fit_transform(temp_data.reshape(-1, 1))

# Prepare training data
X = inputs[:-1]
y = targets[1:]

# ANN model definition
model = Sequential([
    Dense(16, activation='relu', input_shape=(1,)),
    Dense(8, activation='relu'),
    Dense(1)
])
model.compile(optimizer='adam', loss='mse')

# Train ANN
model.fit(X, y, epochs=50, batch_size=32)

# Simulation for ANN controller
control_signal = inputs.copy()
ann_predictions = []

for i in range(len(control_signal)):
    pred = model.predict(control_signal[i].reshape(1, -1))
    ann_predictions.append(pred[0][0])

# PID Controller Simulation
set_point = 0.5
pid_output = []
kp, ki, kd = 0.6, 0.1, 0.05
integral, prev_error = 0, 0

for measurement in targets.flatten():
    error = set_point - measurement
    integral += error
    derivative = error - prev_error
    output = kp * error + ki * integral + kd * derivative
    pid_output.append(output)
    prev_error = error

# Plotting comparison
plt.figure(figsize=(12, 6))
plt.plot(targets, label='Actual Temperature')
plt.plot(ann_predictions, label='ANN Controller Prediction')
plt.plot(pid_output, label='PID Controller Output', alpha=0.7)
plt.title('ANN vs PID Control of Chemical Reactor Temperature')
plt.xlabel('Time Steps')
plt.ylabel('Normalized Temperature')
plt.legend()
plt.grid(True)
plt.show()
