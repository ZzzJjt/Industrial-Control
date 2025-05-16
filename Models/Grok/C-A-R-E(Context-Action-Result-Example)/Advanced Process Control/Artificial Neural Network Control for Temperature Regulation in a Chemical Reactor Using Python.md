import numpy as np
import tensorflow as tf
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense
import matplotlib.pyplot as plt

# Reactor Model (FOPDT with nonlinear dynamics)
class Reactor:
    def __init__(self, dt=1.0):
        self.dt = dt  # Time step (s)
        self.K = 0.8  # Process gain (°C/%)
        self.tau = 100.0  # Time constant (s)
        self.theta = 10.0  # Dead time (s)
        self.T = 20.0  # Current temperature (°C)
        self.T_feed = 20.0  # Feed temperature (°C)
        self.u_history = [0.0] * int(self.theta / dt)  # Control signal history

    def step(self, u, T_feed):
        """Simulate reactor temperature with nonlinear dynamics"""
        self.T_feed = T_feed
        u_delayed = self.u_history.pop(0)  # Get delayed control signal
        self.u_history.append(u)  # Store new control signal

        # Nonlinear gain adjustment (e.g., diminishes at high temperatures)
        K_eff = self.K / (1 + 0.01 * self.T)
        # FOPDT dynamics: dT/dt = -T/tau + K*u_delayed/tau + T_feed/tau
        dT_dt = (-self.T + K_eff * u_delayed + self.T_feed) / self.tau
        self.T += dT_dt * self.dt
        return self.T

# PID Controller
class PID:
    def __init__(self, Kp=2.0, Ki=0.01, Kd=0.5, dt=1.0):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.dt = dt
        self.integral = 0.0
        self.prev_error = 0.0

    def compute(self, setpoint, pv):
        """Compute PID control signal"""
        error = setpoint - pv
        self.integral += error * self.dt
        derivative = (error - self.prev_error) / self.dt
        u = self.Kp * error + self.Ki * self.integral + self.Kd * derivative
        self.prev_error = error
        return np.clip(u, 0, 100)  # Limit to 0–100%

# ANN Controller
class ANNController:
    def __init__(self):
        self.model = self.build_model()
    
    def build_model(self):
        """Build ANN model"""
        model = Sequential([
            Dense(10, activation='relu', input_shape=(3,)),  # Input: T, SP, T_feed
            Dense(10, activation='relu'),
            Dense(1, activation='sigmoid')  # Output: u (0–1, scaled to 0–100%)
        ])
        model.compile(optimizer='adam', loss='mse')
        return model

    def train(self, X, y, epochs=50):
        """Train ANN on historical data"""
        self.model.fit(X, y, epochs=epochs, verbose=0)

    def predict(self, T, SP, T_feed):
        """Predict control signal"""
        X = np.array([[T, SP, T_feed]])
        u = self.model.predict(X, verbose=0)[0, 0] * 100  # Scale to 0–100%
        return np.clip(u, 0, 100)

# Generate Historical Data
def generate_data(reactor, pid, n_samples=10000):
    """Generate synthetic process data for ANN training"""
    T_data, SP_data, T_feed_data, u_data = [], [], [], []
    T = 20.0
    SP = 75.0
    T_feed = 20.0

    for _ in range(n_samples):
        # Randomize setpoint and feed temperature periodically
        if _ % 1000 == 0:
            SP = np.random.uniform(70, 80)
            T_feed = np.random.uniform(15, 25)
        
        # Use PID to generate control signal
        u = pid.compute(SP, T)
        T_data.append(T)
        SP_data.append(SP)
        T_feed_data.append(T_feed)
        u_data.append(u)
        T = reactor.step(u, T_feed)

    X = np.array([T_data, SP_data, T_feed_data]).T
    y = np.array(u_data) / 100  # Normalize to 0–1
    return X, y

# Simulate Control Performance
def simulate(reactor, ann, pid, t_sim=2000, dt=1.0):
    """Simulate ANN and PID control with feed temperature disturbance"""
    t = np.arange(0, t_sim, dt)
    T_ann, u_ann, T_pid, u_pid = [], [], [], []
    SP = 75.0
    T_feed = [20.0 if ti < 1000 else 30.0 for ti in t]  # Disturbance at t=1000s
    
    # Reset reactor and PID
    reactor.T = 20.0
    reactor.u_history = [0.0] * int(reactor.theta / dt)
    pid.integral = 0.0
    pid.prev_error = 0.0
    
    # ANN simulation
    for i in range(len(t)):
        u = ann.predict(reactor.T, SP, T_feed[i])
        T = reactor.step(u, T_feed[i])
        T_ann.append(T)
        u_ann.append(u)
    
    # Reset reactor for PID simulation
    reactor.T = 20.0
    reactor.u_history = [0.0] * int(reactor.theta / dt)
    
    # PID simulation
    for i in range(len(t)):
        u = pid.compute(SP, reactor.T)
        T = reactor.step(u, T_feed[i])
        T_pid.append(T)
        u_pid.append(u)
    
    return t, T_ann, u_ann, T_pid, u_pid, SP, T_feed

# Plot Results
def plot_results(t, T_ann, u_ann, T_pid, u_pid, SP, T_feed):
    """Plot ANN vs PID performance"""
    plt.figure(figsize=(12, 10))
    
    # Temperature
    plt.subplot(3, 1, 1)
    plt.plot(t, T_ann, label='ANN')
    plt.plot(t, T_pid, label='PID')
    plt.axhline(SP, color='r', linestyle='--', label='Setpoint')
    plt.ylabel('Temperature (°C)')
    plt.title('Reactor Temperature Control')
    plt.legend()
    
    # Control Signal
    plt.subplot(3, 1, 2)
    plt.plot(t, u_ann, label='ANN')
    plt.plot(t, u_pid, label='PID')
    plt.ylabel('Control Signal (%)')
    plt.title('Control Signal')
    plt.legend()
    
    # Feed Temperature
    plt.subplot(3, 1, 3)
    plt.plot(t, T_feed, label='Feed Temperature')
    plt.ylabel('Feed Temperature (°C)')
    plt.xlabel('Time (s)')
    plt.title('Feed Temperature Disturbance')
    plt.legend()
    
    plt.tight_layout()
    plt.savefig('reactor_control.png')
    plt.show()

# Main Execution
if __name__ == "__main__":
    # Initialize components
    reactor = Reactor(dt=1.0)
    pid = PID(Kp=2.0, Ki=0.01, Kd=0.5, dt=1.0)
    ann = ANNController()
    
    # Generate and train on historical data
    X, y = generate_data(reactor, pid, n_samples=10000)
    ann.train(X, y, epochs=50)
    
    # Simulate control performance
    t, T_ann, u_ann, T_pid, u_pid, SP, T_feed = simulate(reactor, ann, pid, t_sim=2000)
    
    # Plot results
    plot_results(t, T_ann, u_ann, T_pid, u_pid, SP, T_feed)
    
    # Calculate performance metrics
    mse_ann = np.mean((np.array(T_ann) - SP) ** 2)
    mse_pid = np.mean((np.array(T_pid) - SP) ** 2)
    energy_ann = np.sum(np.array(u_ann)) * reactor.dt / 3600  # kWh (assuming 1% = 1 kW)
    energy_pid = np.sum(np.array(u_pid)) * reactor.dt / 3600  # kWh
    
    print(f"ANN MSE: {mse_ann:.2f} °C²")
    print(f"PID MSE: {mse_pid:.2f} °C²")
    print(f"ANN Energy: {energy_ann:.2f} kWh")
    print(f"PID Energy: {energy_pid:.2f} kWh")
