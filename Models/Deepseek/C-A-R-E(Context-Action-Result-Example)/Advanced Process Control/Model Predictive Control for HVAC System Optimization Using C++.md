import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint

# Process parameters
feed_rate = 50  # tons/hour
tank_capacity = 1000  # m^3
upstream_delay = 2 * 3600  # seconds (2 hours)

# Simulation settings
time_end = 48 * 3600  # seconds (48 hours)
dt = 60  # seconds (1 minute)
time_points = np.arange(0, time_end + dt, dt)

# Demand profile
demand_base = 45  # tons/hour
demand_spike_start = 18 * 3600  # seconds (18 hours)
demand_spike_duration = 6 * 3600  # seconds (6 hours)
demand_spike_magnitude = 60  # tons/hour

def demand_profile(t):
    if demand_spike_start <= t < demand_spike_start + demand_spike_duration:
        return demand_spike_magnitude
    else:
        return demand_base

# Tank dynamics model
def tank_dynamics(y, t, u):
    dhdt = (u - demand_profile(t)) / tank_capacity
    return dhdt

# PID Controller
class PIDController:
    def __init__(self, Kp, Ki, Kd, setpoint):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.setpoint = setpoint
        self.error_integral = 0
        self.last_error = 0

    def update(self, measured_value, dt):
        error = self.setpoint - measured_value
        self.error_integral += error * dt
        derivative = (error - self.last_error) / dt
        output = self.Kp * error + self.Ki * self.error_integral + self.Kd * derivative
        self.last_error = error
        return output

# MPC Controller
class MPCController:
    def __init__(self, horizon, timestep, setpoint):
        self.horizon = horizon
        self.timestep = timestep
        self.setpoint = setpoint

    def predict(self, current_level, u_prev):
        predictions = []
        h = current_level
        for _ in range(self.horizon):
            h = odeint(tank_dynamics, h, [0, self.timestep], args=(u_prev[-1],))[1][0]
            predictions.append(h)
        return predictions

    def calculate_control(self, current_level, u_prev):
        cost_min = float('inf')
        best_u = None
        for u in np.linspace(demand_base, feed_rate, 10):  # Discretize possible inputs
            predictions = self.predict(current_level, u_prev + [u])
            cost = sum((h - self.setpoint) ** 2 for h in predictions)
            if cost < cost_min:
                cost_min = cost
                best_u = u
        return best_u

# Initialize controllers
pid_controller = PIDController(Kp=1.0, Ki=0.1, Kd=0.01, setpoint=0.7 * tank_capacity)
mpc_controller = MPCController(horizon=int(upstream_delay / dt), timestep=dt, setpoint=0.7 * tank_capacity)

# Simulation loop
h_pid = [0.7 * tank_capacity]  # Initial tank level
h_mpc = [0.7 * tank_capacity]  # Initial tank level
u_pid = []
u_mpc = []

for i in range(1, len(time_points)):
    t = time_points[i]

    # PID control action
    pid_output = pid_controller.update(h_pid[-1], dt)
    u_pid_next = max(min(pid_output + demand_profile(t), feed_rate), demand_base)
    u_pid.append(u_pid_next)
    
    # Simulate delay in input for PID
    if i >= int(upstream_delay / dt):
        u_pid_real = u_pid[i - int(upstream_delay / dt)]
    else:
        u_pid_real = demand_profile(t)

    # Update tank level using ODE solver for PID
    new_h_pid = odeint(tank_dynamics, h_pid[-1], [0, dt], args=(u_pid_real,))[1][0]
    h_pid.append(new_h_pid)

    # MPC control action
    mpc_output = mpc_controller.calculate_control(h_mpc[-1], u_mpc)
    u_mpc.append(mpc_output)
    
    # Simulate delay in input for MPC
    if i >= int(upstream_delay / dt):
        u_mpc_real = u_mpc[i - int(upstream_delay / dt)]
    else:
        u_mpc_real = demand_profile(t)

    # Update tank level using ODE solver for MPC
    new_h_mpc = odeint(tank_dynamics, h_mpc[-1], [0, dt], args=(u_mpc_real,))[1][0]
    h_mpc.append(new_h_mpc)

# Plot results
plt.figure(figsize=(14, 7))
plt.plot(time_points / 3600, h_pid, label='PID Controlled Level', linewidth=2)
plt.plot(time_points / 3600, h_mpc, label='MPC Controlled Level', linewidth=2)
plt.axhline(0.7 * tank_capacity, color='r', linestyle='--', label='Setpoint')
plt.xlabel('Time (hours)')
plt.ylabel('Tank Level (m³)')
plt.title('Buffer Tank Levels with PID and MPC Controllers')
plt.legend()
plt.grid(True)
plt.show()



