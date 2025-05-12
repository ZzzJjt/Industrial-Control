import numpy as np
import tensorflow as tf
from tensorflow.keras import layers
import matplotlib.pyplot as plt
import scipy.integrate

# Set random seed for reproducibility
np.random.seed(42)
tf.random.set_seed(42)

# Reactor simulation parameters
class ReactorModel:
    def __init__(self):
        self.T0 = 300  # Initial temperature (K)
        self.Q = 1000  # Heat input (W)
        self.Cp = 4.18  # Heat capacity (J/gK)
        self.m = 1000  # Mass (g)
        self.UA = 50   # Heat transfer coefficient (W/K)
        self.T_amb = 298  # Ambient temperature (K)
        
    def dynamics(self, t, T, Q):
        # Nonlinear reactor dynamics
        dTdt = (Q - self.UA * (T - self.T_amb)) / (self.m * self.Cp)
        # Add nonlinear term
        dTdt += -0.001 * (T - self.T0)**2
        return dTdt
    
    def simulate(self, t_span, Q, T_init):
        sol = scipy.integrate.solve_ivp(
            fun=lambda t, T: self.dynamics(t, T, Q),
            t_span=(t_span[0], t_span[-1]),
            y0=[T_init],
            t_eval=t_span
        )
        return sol.y[0]

# PID Controller
class PIDController:
    def __init__(self, Kp=10.0, Ki=0.1, Kd=0.05):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.prev_error = 0
        self.integral = 0
        
    def control(self, setpoint, measured, dt):
        error = setpoint - measured
        self.integral += error * dt
        derivative = (error - self.prev_error) / dt
        output = self.Kp * error + self.Ki * self.integral + self.Kd * derivative
        self.prev_error = error
        return np.clip(output, 0, 2000)  # Limit heat input

# ANN Controller
class ANNController:
    def __init__(self):
        self.model = self.build_model()
        
    def build_model(self):
        model = tf.keras.Sequential([
            layers.Dense(64, activation='relu', input_shape=(3,)),
            layers.Dense(32, activation='relu'),
            layers.Dense(1, activation='linear')
        ])
        model.compile(optimizer='adam', loss='mse')
        return model
    
    def train(self, X, y, epochs=100):
        self.model.fit(X, y, epochs=epochs, verbose=0)
        
    def control(self, setpoint, measured, time):
        inputs = np.array([[setpoint, measured, time]])
        return np.clip(self.model.predict(inputs, verbose=0)[0][0], 0, 2000)

# Generate training data
def generate_training_data(reactor, pid, t_span, setpoint):
    t = np.linspace(t_span[0], t_span[1], len(t_span))
    T = np.zeros_like(t)
    Q = np.zeros_like(t)
    T[0] = reactor.T0
    
    # Simulate with PID to generate training data
    for i in range(1, len(t)):
        dt = t[i] - t[i-1]
        Q[i-1] = pid.control(setpoint(t[i-1]), T[i-1], dt)
        T[i] = reactor.simulate([t[i-1], t[i]], Q[i-1], T[i-1])[-1]
    
    # Prepare ANN training data
    X = np.column_stack([setpoint(t), T, t])
    y = Q.reshape(-1, 1)
    return X, y, T

# Simulation and comparison
def run_simulation():
    reactor = ReactorModel()
    pid = PIDController()
    ann = ANNController()
    
    # Time span
    t_span = np.linspace(0, 100, 1000)
    
    # Varying setpoint with disturbances
    def setpoint(t):
        return 350 + 10 * np.sin(0.1 * t) + 5 * np.random.normal(0, 1)
    
    # Generate training data
    X, y, _ = generate_training_data(reactor, pid, t_span, setpoint)
    ann.train(X, y)
    
    # Simulate both controllers
    T_pid = np.zeros_like(t_span)
    T_ann = np.zeros_like(t_span)
    Q_pid = np.zeros_like(t_span)
    Q_ann = np.zeros_like(t_span)
    
    T_pid[0] = reactor.T0
    T_ann[0] = reactor.T0
    
    for i in range(1, len(t_span)):
        dt = t_span[i] - t_span[i-1]
        
        # PID control
        Q_pid[i-1] = pid.control(setpoint(t_span[i-1]), T_pid[i-1], dt)
        T_pid[i] = reactor.simulate([t_span[i-1], t_span[i]], Q_pid[i-1], T_pid[i-1])[-1]
        
        # ANN control
        Q_ann[i-1] = ann.control(setpoint(t_span[i-1]), T_ann[i-1], t_span[i-1])
        T_ann[i] = reactor.simulate([t_span[i-1], t_span[i]], Q_ann[i-1], T_ann[i-1])[-1]
    
    # Calculate performance metrics
    error_pid = np.abs(setpoint(t_span) - T_pid)
    error_ann = np.abs(setpoint(t_span) - T_ann)
    
    # Plot results
    plt.figure(figsize=(12, 8))
    
    plt.subplot(2, 1, 1)
    plt.plot(t_span, setpoint(t_span), 'k--', label='Setpoint')
    plt.plot(t_span, T_pid, 'b-', label='PID')
    plt.plot(t_span, T_ann, 'r-', label='ANN')
    plt.xlabel('Time (s)')
    plt.ylabel('Temperature (K)')
    plt.title('Temperature Control Comparison')
    plt.legend()
    plt.grid(True)
    
    plt.subplot(2, 1, 2)
    plt.plot(t_span, error_pid, 'b-', label='PID Error')
    plt.plot(t_span, error_ann, 'r-', label='ANN Error')
    plt.xlabel('Time (s)')
    plt.ylabel('Absolute Error (K)')
    plt.title('Control Error Comparison')
    plt.legend()
    plt.grid(True)
    
    plt.tight_layout()
    plt.savefig('reactor_control_comparison.png')
    
    # Print performance metrics
    print(f"PID Mean Absolute Error: {np.mean(error
