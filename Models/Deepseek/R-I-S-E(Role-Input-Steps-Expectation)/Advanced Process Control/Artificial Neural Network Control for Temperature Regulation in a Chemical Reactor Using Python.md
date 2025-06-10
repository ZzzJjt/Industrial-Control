import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense

# Load dataset
data = pd.read_csv('reactor_data.csv')

# Assume columns are known
X = data[['coolant_flow', 'feed_temperature', 'reactant_concentration']]
y = data['reactor_temperature']

# Split and scale data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# Build ANN model
model = Sequential([
    Dense(64, activation='relu', input_shape=(X_train_scaled.shape[1],)),
    Dense(64, activation='relu'),
    Dense(1)  # Predict temperature
])

model.compile(optimizer='adam', loss='mse')
model.fit(X_train_scaled, y_train, epochs=50, batch_size=32, validation_split=0.1)

# Evaluate
loss = model.evaluate(X_test_scaled, y_test)
print(f"Test Loss (MSE): {loss}")

def ann_controller_step(model, scaler, current_inputs, setpoint):
    """
    Use ANN model to predict temperature and adjust coolant flow.
    
    Args:
        model: Trained Keras model
        scaler: Scaler used during training
        current_inputs: dict with keys ['coolant_flow', 'feed_temperature', 'reactant_concentration']
        setpoint: float, desired temperature
    
    Returns:
        new_coolant_flow: adjusted value
    """
    input_array = np.array([[
        current_inputs['coolant_flow'],
        current_inputs['feed_temperature'],
        current_inputs['reactant_concentration']
    ]])
    scaled_input = scaler.transform(input_array)
    predicted_temp = model.predict(scaled_input, verbose=0)[0][0]
    
    error = setpoint - predicted_temp
    
    # Tune gain manually or use adaptive tuning later
    Kp_ann = 0.5  
    delta_coolant = Kp_ann * error
    new_coolant_flow = np.clip(current_inputs['coolant_flow'] + delta_coolant, 0, 100)  # Limit range
    
    return new_coolant_flow, predicted_temp


    import matplotlib.pyplot as plt

# Simulate dynamics over time
time_steps = 100
setpoint = 85.0
initial_inputs = {
    'coolant_flow': 50.0,
    'feed_temperature': 70.0,
    'reactant_concentration': 1.0
}

# Dummy function simulating real reactor behavior (could be replaced with a first-principles model)
def simulate_reactor_dynamics(inputs):
    # Simplified nonlinear relationship
    coolant_flow = inputs['coolant_flow']
    feed_temp = inputs['feed_temperature']
    conc = inputs['reactant_concentration']
    temp = 0.6 * feed_temp + 0.3 * conc * 100 - 0.1 * coolant_flow
    return temp + np.random.normal(0, 0.5)  # Add noise

# Initialize arrays
times = np.arange(time_steps)
temps_ann = []
temps_pid = []

# PID parameters
Kp, Ki, Kd = 2.0, 0.1, 0.05
integral = 0
prev_error = 0

# Initial states
inputs_ann = initial_inputs.copy()
inputs_pid = initial_inputs.copy()

for t in times:
    # ANN Controller Loop
    inputs_ann['coolant_flow'], pred_temp_ann = ann_controller_step(
        model, scaler, inputs_ann, setpoint
    )
    actual_temp_ann = simulate_reactor_dynamics(inputs_ann)
    temps_ann.append(actual_temp_ann)
    
    # PID Controller Loop
    actual_temp_pid = simulate_reactor_dynamics(inputs_pid)
    error = setpoint - actual_temp_pid
    integral += error
    derivative = error - prev_error
    delta_coolant_pid = Kp * error + Ki * integral + Kd * derivative
    inputs_pid['coolant_flow'] = np.clip(inputs_pid['coolant_flow'] + delta_coolant_pid, 0, 100)
    prev_error = error
    temps_pid.append(actual_temp_pid)


  plt.figure(figsize=(12, 6))
plt.plot(times, temps_ann, label='ANN Controller', linestyle='-', color='blue')
plt.plot(times, temps_pid, label='PID Controller', linestyle='--', color='orange')
plt.axhline(y=setpoint, color='green', linestyle=':', label='Setpoint')
plt.xlabel('Time Step')
plt.ylabel('Reactor Temperature (°C)')
plt.title('Temperature Control Comparison: ANN vs PID')
plt.legend()
plt.grid(True)
plt.show()
