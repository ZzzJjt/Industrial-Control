import numpy as np
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
import matplotlib.pyplot as plt
from scipy.integrate import solve_ivp

# Step 1: Data Preprocessing
# Load historical process data (simulated here for demonstration)
np.random.seed(42)
time = np.linspace(0, 100, 1000)
setpoints = np.sin(time * 0.1) * 10 + 30  # Time-varying setpoint
noise = np.random.normal(0, 1, time.shape)
temperatures = setpoints + noise  # Simulated reactor temperatures with noise

data = pd.DataFrame({'Time': time, 'Setpoint': setpoints, 'Temperature': temperatures})

# Normalize the data
scaler = StandardScaler()
data[['Setpoint', 'Temperature']] = scaler.fit_transform(data[['Setpoint', 'Temperature']])

# Prepare input-output pairs for training
X = data[['Setpoint']].values
y = data[['Temperature']].values

# Split the data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Step 2: Model Training
# Build a simple feedforward neural network
model = Sequential([
    Dense(64, activation='relu', input_shape=(1,)),
    Dense(64, activation='relu'),
    Dense(1)
])

model.compile(optimizer='adam', loss='mse')

# Train the model
history = model.fit(X_train, y_train, epochs=100, batch_size=32, validation_split=0.2, verbose=1)

# Step 3: Control Logic Implementation
def nn_controller(setpoint):
    """Neural Network Controller"""
    scaled_setpoint = scaler.transform([[setpoint]])
    predicted_temperature = model.predict(scaled_setpoint)
    return predicted_temperature[0][0]

def pid_controller(setpoint, current_temp, prev_error, integral, dt, Kp, Ki, Kd):
    """PID Controller"""
    error = setpoint - current_temp
    integral += error * dt
    derivative = (error - prev_error) / dt
    output = Kp * error + Ki * integral + Kd * derivative
    return output, error, integral

# Parameters for PID controller
Kp = 1.0
Ki = 0.1
Kd = 0.01

# Simulation parameters
dt = 0.1
total_time = 100
num_steps = int(total_time / dt)
time_points = np.linspace(0, total_time, num_steps)

# Initial conditions
current_temp = 30.0
prev_error = 0.0
integral = 0.0

# Arrays to store results
nn_temps = []
pid_temps = []
setpoints = []

# Step 4: Simulation
for t in time_points:
    setpoint = np.sin(t * 0.1) * 10 + 30
    
    # NN Controller
    nn_output = nn_controller(setpoint)
    nn_temps.append(nn_output)
    
    # PID Controller
    pid_output, prev_error, integral = pid_controller(setpoint, current_temp, prev_error, integral, dt, Kp, Ki, Kd)
    current_temp += pid_output * dt  # Simple first-order approximation of reactor dynamics
    pid_temps.append(current_temp)
    
    setpoints.append(setpoint)

# Convert lists to arrays for plotting
nn_temps = np.array(nn_temps)
pid_temps = np.array(pid_temps)
setpoints = np.array(setpoints)

# Step 5: Result Visualization
plt.figure(figsize=(14, 7))

plt.plot(time_points, setpoints, label='Setpoint', linestyle='--')
plt.plot(time_points, nn_temps, label='NN Controlled Temperature', linewidth=2)
plt.plot(time_points, pid_temps, label='PID Controlled Temperature', linewidth=2)

plt.xlabel('Time [s]')
plt.ylabel('Temperature [°C]')
plt.title('Comparison of NN and PID Controllers for Chemical Reactor Temperature Control')
plt.legend()
plt.grid(True)
plt.show()
