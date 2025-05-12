import numpy as np
import pandas as pd
import tensorflow as tf
from sklearn.preprocessing import MinMaxScaler
from sklearn.model_selection import train_test_split
from matplotlib import pyplot as plt

# 1. Data Preparation
def load_and_preprocess_data():
    # Load historical process data (simulated example)
    # Columns: [time, temp_in, coolant_flow, pressure, temp_out]
    data = pd.read_csv('reactor_history.csv')
    
    # Feature engineering
    X = data[['temp_in', 'coolant_flow', 'pressure']].values
    y = data['temp_out'].values.reshape(-1,1)
    
    # Normalize data
    x_scaler = MinMaxScaler()
    y_scaler = MinMaxScaler()
    X_scaled = x_scaler.fit_transform(X)
    y_scaled = y_scaler.fit_transform(y)
    
    return train_test_split(X_scaled, y_scaled, test_size=0.2, shuffle=False)

# 2. ANN Model Development
def build_ann_model(input_shape):
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(64, activation='relu', input_shape=input_shape),
        tf.keras.layers.Dropout(0.2),
        tf.keras.layers.Dense(32, activation='relu'),
        tf.keras.layers.Dense(1)
    ])
    
    model.compile(optimizer='adam',
                 loss='mse',
                 metrics=['mae'])
    return model

# 3. Simulation Environment
class ReactorSimulator:
    def __init__(self, ann_model, x_scaler, y_scaler):
        self.model = ann_model
        self.x_scaler = x_scaler
        self.y_scaler = y_scaler
        self.history = []
        
    def step(self, current_state, set_point):
        # Prepare input
        nn_input = np.array([current_state['temp'], 
                            current_state['coolant'], 
                            current_state['pressure']]).reshape(1,-1)
        nn_input = self.x_scaler.transform(nn_input)
        
        # Predict next state
        predicted_temp = self.model.predict(nn_input, verbose=0)
        predicted_temp = self.y_scaler.inverse_transform(predicted_temp)[0][0]
        
        # Simple PID for comparison
        error = set_point - current_state['temp']
        pid_output = 0.5*error + 0.1*(error - current_state.get('last_error', 0))
        
        # Store results
        self.history.append({
            'time': current_state['time'],
            'ann_temp': predicted_temp,
            'pid_temp': current_state['temp'] + pid_output,
            'set_point': set_point
        })
        
        return predicted_temp

# 4. Performance Comparison
def compare_performance(simulator):
    plt.figure(figsize=(12,6))
    history = pd.DataFrame(simulator.history)
    plt.plot(history['time'], history['ann_temp'], label='ANN Control')
    plt.plot(history['time'], history['pid_temp'], label='PID Control')
    plt.plot(history['time'], history['set_point'], 'k--', label='Set Point')
    plt.xlabel('Time (min)')
    plt.ylabel('Temperature (°C)')
    plt.title('Reactor Temperature Control Performance')
    plt.legend()
    plt.grid(True)
    
    # Calculate metrics
    ann_error = np.abs(history['ann_temp'] - history['set_point'])
    pid_error = np.abs(history['pid_temp'] - history['set_point'])
    
    print(f"ANN Mean Absolute Error: {np.mean(ann_error):.2f}°C")
    print(f"PID Mean Absolute Error: {np.mean(pid_error):.2f}°C")
    print(f"Improvement: {(1 - np.mean(ann_error)/np.mean(pid_error))*100:.1f}%")

# Main execution
if __name__ == "__main__":
    # Prepare data and model
    X_train, X_test, y_train, y_test = load_and_preprocess_data()
    model = build_ann_model((X_train.shape[1],))
    model.fit(X_train, y_train, epochs=100, batch_size=32, validation_split=0.1)
    
    # Initialize simulator
    simulator = ReactorSimulator(model, x_scaler, y_scaler)
    
    # Run simulation with time-varying setpoints
    current_state = {'temp': 150, 'coolant': 20, 'pressure': 1.0, 'time': 0}
    for t in range(1, 121):  # 2 hours simulation
        current_state['time'] = t
        # Varying setpoint to simulate real conditions
        set_point = 150 + 10*np.sin(t/10) + 5*(t%20>10)
        
        # ANN control step
        new_temp = simulator.step(current_state, set_point)
        
        # Update state (simplified dynamics)
        current_state['temp'] = new_temp * (0.95 + 0.05*np.random.randn())
        current_state['coolant'] = max(10, min(30, current_state['coolant'] + np.random.randint(-1,2)))
        current_state['pressure'] = max(0.8, min(1.2, current_state['pressure'] + 0.05*np.random.randn()))
    
    # Compare performance
    compare_performance(simulator)
