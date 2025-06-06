import numpy as np
import pandas as pd
from sklearn.preprocessing import MinMaxScaler
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
from tensorflow.keras.optimizers import Adam

# Load historical dataset
data = pd.read_csv('reactor_temp_data.csv')  # Columns: ['T_in', 'Q', 'T_out']
scaler = MinMaxScaler()
data_scaled = scaler.fit_transform(data)

X = data_scaled[:, :2]   # Features: [T_in, Q]
y = data_scaled[:, 2]    # Target: T_out

# Build ANN model
ann_model = Sequential([
    Dense(64, input_dim=2, activation='relu'),
    Dense(64, activation='relu'),
    Dense(1, activation='linear')
])
ann_model.compile(optimizer=Adam(0.001), loss='mse')

# Train model
ann_model.fit(X, y, epochs=100, batch_size=32, verbose=0)

def ann_temperature_controller(T_set, T_in, T_prev, ann_model, scaler):
    Q_space = np.linspace(0, 1, 100)  # Control input normalized
    best_Q = 0
    min_error = float('inf')
    for Q in Q_space:
        X_pred = scaler.transform([[T_in, Q]])[:, :2]
        T_pred = ann_model.predict(X_pred, verbose=0)[0][0]
        err = abs(T_pred - scaler.transform([[0, 0, T_set]])[0][2])
        if err < min_error:
            min_error = err
            best_Q = Q
    return scaler.inverse_transform([[0, best_Q, 0]])[0][1]
