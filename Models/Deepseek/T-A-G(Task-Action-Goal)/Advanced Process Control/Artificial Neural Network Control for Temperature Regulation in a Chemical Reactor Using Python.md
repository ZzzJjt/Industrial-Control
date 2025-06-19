import numpy as np
import matplotlib.pyplot as plt
from sklearn.neural_network import MLPRegressor
from scipy.integrate import solve_ivp

# Step 1: Model the Chemical Reactor Dynamics
def reactor_dynamics(t, y, u):
    # Simplified nonlinear and time-varying dynamics
    k = 0.5 + 0.1 * np.sin(t)
    dydt = -k * y + u
    return dydt

# Step 2: Generate Historical Data
def generate_data(num_samples=1000, noise_level=0.1):
    t_span = (0, 10)
    t_eval = np.linspace(*t_span, num_samples)
    u = np.random.uniform(-1, 1, size=num_samples)  # Control input
    sol = solve_ivp(fun=lambda t, y: reactor_dynamics(t, y, u[int(t / (t_span[1] / num_samples))]), 
                    t_span=t_span, y0=[0], t_eval=t_eval)
    y = sol.y.flatten()
    y_noisy = y + noise_level * np.random.randn(num_samples)
    return t_eval, y_noisy, u

# Generate training data
t_train, y_train, u_train = generate_data()

# Step 3: Design the ANN Controller
ann_controller = MLPRegressor(hidden_layer_sizes=(20, 20), activation='relu', solver='adam', max_iter=1000)

# Step 4: Train the ANN
X_train = np.column_stack((y_train[:-1], u_train[:-1]))
y_next = y_train[1:]
ann_controller.fit(X_train, y_next)

# Function to predict next state using ANN
def ann_predict(y_prev, u_prev):
    X_pred = np.array([[y_prev, u_prev]])
    y_pred = ann_controller.predict(X_pred)[0]
    return y_pred

# Step 5: Simulate Temperature Response
def simulate_control(controller, setpoint, initial_condition, t_span, dt):
    t_eval = np.arange(*t_span, dt)
    y = [initial_condition]
    u = []
    
    for t in t_eval:
        error = setpoint - y[-1]
        if controller == 'PID':
            Kp, Ki, Kd = 1.0, 0.1, 0.05
            integral += error * dt
            derivative = (error - prev_error) / dt
            u_t = Kp * error + Ki * integral + Kd * derivative
            prev_error = error
        elif controller == 'ANN':
            if len(u) > 0:
                u_t = ann_predict(y[-1], u[-1])
            else:
                u_t = 0
        else:
            raise ValueError("Unknown controller type")
        
        sol = solve_ivp(fun=lambda t, y: reactor_dynamics(t, y, u_t), 
                        t_span=(t, t + dt), y0=[y[-1]], t_eval=[t + dt])
        y.append(sol.y.flatten()[-1])
        u.append(u_t)
    
    return t_eval, y, u

# Simulation parameters
setpoint = 1.0
initial_condition = 0.0
t_span = (0, 10)
dt = 0.1

# Initialize variables for PID simulation
integral = 0
prev_error = 0

# Run simulations
t_pid, y_pid, u_pid = simulate_control('PID', setpoint, initial_condition, t_span, dt)
t_ann, y_ann, u_ann = simulate_control('ANN', setpoint, initial_condition, t_span, dt)

# Plot results
plt.figure(figsize=(12, 6))

plt.subplot(2, 1, 1)
plt.plot(t_pid, y_pid, label='PID Controlled')
plt.plot(t_ann, y_ann, label='ANN Controlled')
plt.axhline(setpoint, color='r', linestyle='--', label='Setpoint')
plt.title('Temperature Response')
plt.xlabel('Time')
plt.ylabel('Temperature')
plt.legend()

plt.subplot(2, 1, 2)
plt.plot(t_pid, u_pid, label='PID Control Signal')
plt.plot(t_ann, u_ann, label='ANN Control Signal')
plt.title('Control Signals')
plt.xlabel('Time')
plt.ylabel('Control Signal')
plt.legend()

plt.tight_layout()
plt.show()



