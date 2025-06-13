import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize
from scipy.interpolate import interp1d

# Step 1: Simulate System Dynamics
def simulate_system(infeed_rates, demand, delay_hours=2):
    dt = 1 / 60  # Time step in hours (1 minute)
    total_time = len(demand) * dt
    t = np.arange(0, total_time, dt)
    
    buffer_level = np.zeros_like(t)
    outfeed_rate = np.zeros_like(t)
    
    for i in range(len(t)):
        if i >= int(delay_hours / dt):
            outfeed_rate[i] = demand[i - int(delay_hours / dt)]
        else:
            outfeed_rate[i] = 0
        
        if i > 0:
            buffer_level[i] = buffer_level[i - 1] + infeed_rates[i - 1] * dt - outfeed_rate[i] * dt
    
    return buffer_level, outfeed_rate

# Generate synthetic demand data with fluctuations
np.random.seed(0)
num_steps = 120  # 2 hours of simulation at 1-minute intervals
demand = 5 + 2 * np.sin(np.linspace(0, 4 * np.pi, num_steps)) + np.random.normal(0, 0.5, num_steps)

# Step 2: Design the MPC Controller
class MPCController:
    def __init__(self, horizon=60, delay_hours=2):
        self.horizon = horizon
        self.delay_hours = delay_hours
        self.dt = 1 / 60  # Time step in hours (1 minute)
    
    def predict(self, current_buffer, infeed_rates, demand_forecast):
        predicted_buffer_levels = np.zeros(self.horizon)
        predicted_outfeed_rates = np.zeros(self.horizon)
        
        for i in range(self.horizon):
            if i >= int(self.delay_hours / self.dt):
                predicted_outfeed_rates[i] = demand_forecast[i - int(self.delay_hours / self.dt)]
            else:
                predicted_outfeed_rates[i] = 0
            
            if i == 0:
                predicted_buffer_levels[i] = current_buffer + infeed_rates[i] * self.dt - predicted_outfeed_rates[i] * self.dt
            else:
                predicted_buffer_levels[i] = predicted_buffer_levels[i - 1] + infeed_rates[i] * self.dt - predicted_outfeed_rates[i] * self.dt
        
        return predicted_buffer_levels, predicted_outfeed_rates
    
    def objective_function(self, infeed_rates, current_buffer, demand_forecast):
        predicted_buffer_levels, _ = self.predict(current_buffer, infeed_rates, demand_forecast)
        cost = np.sum((predicted_buffer_levels - 10)**2)  # Target buffer level is 10 units
        return cost
    
    def optimize(self, current_buffer, demand_forecast):
        initial_guess = np.ones(self.horizon) * 10  # Initial guess for infeed rates
        bounds = [(0, 20)] * self.horizon  # Bounds for infeed rates
        
        result = minimize(self.objective_function, initial_guess, args=(current_buffer, demand_forecast), method='SLSQP', bounds=bounds)
        optimized_infeed_rates = result.x
        return optimized_infeed_rates

# Step 3: Validate Performance
mpc_controller = MPCController(horizon=60, delay_hours=2)
pid_controller = lambda buffer_level, target=10, Kp=1, Ki=0.1, Kd=0.05: Kp * (target - buffer_level) + Ki * integral + Kd * derivative

buffer_level_mpc = np.zeros(num_steps)
buffer_level_pid = np.zeros(num_steps)
outfeed_rate_mpc = np.zeros(num_steps)
outfeed_rate_pid = np.zeros(num_steps)
infeed_rate_mpc = np.zeros(num_steps)
infeed_rate_pid = np.zeros(num_steps)

integral_mpc = 0
integral_pid = 0
prev_error_mpc = 0
prev_error_pid = 0

for i in range(num_steps):
    # Forecast demand for the next horizon
    demand_forecast = demand[max(0, i):i + mpc_controller.horizon]
    if len(demand_forecast) < mpc_controller.horizon:
        demand_forecast = np.pad(demand_forecast, (0, mpc_controller.horizon - len(demand_forecast)), mode='edge')
    
    # MPC Control
    optimized_infeed_rates = mpc_controller.optimize(buffer_level_mpc[i], demand_forecast)
    infeed_rate_mpc[i] = optimized_infeed_rates[0]
    
    # Update buffer level and outfeed rate for MPC
    buffer_level_mpc[i+1:i+int(mpc_controller.delay_hours/mpc_controller.dt)+1], outfeed_rate_mpc[i:i+int(mpc_controller.delay_hours/mpc_controller.dt)+1] = \
        simulate_system([infeed_rate_mpc[i]] * int(mpc_controller.delay_hours/mpc_controller.dt), demand[i:i+int(mpc_controller.delay_hours/mpc_controller.dt)], delay_hours=mpc_controller.delay_hours)
    
    # PID Control
    error_pid = 10 - buffer_level_pid[i]
    integral_pid += error_pid * mpc_controller.dt
    derivative_pid = (error_pid - prev_error_pid) / mpc_controller.dt
    infeed_rate_pid[i] = pid_controller(buffer_level_pid[i], target=10, Kp=1, Ki=0.1, Kd=0.05)
    prev_error_pid = error_pid
    
    # Update buffer level and outfeed rate for PID
    buffer_level_pid[i+1:i+int(mpc_controller.delay_hours/mpc_controller.dt)+1], outfeed_rate_pid[i:i+int(mpc_controller.delay_hours/mpc_controller.dt)+1] = \
        simulate_system([infeed_rate_pid[i]] * int(mpc_controller.delay_hours/mpc_controller.dt), demand[i:i+int(mpc_controller.delay_hours/mpc_controller.dt)], delay_hours=mpc_controller.delay_hours)

# Plot results
plt.figure(figsize=(12, 8))

plt.subplot(2, 1, 1)
plt.plot(buffer_level_mpc[:-int(mpc_controller.delay_hours/mpc_controller.dt)-1], label='Buffer Level (MPC)')
plt.plot(buffer_level_pid[:-int(mpc_controller.delay_hours/mpc_controller.dt)-1], label='Buffer Level (PID)')
plt.axhline(10, color='r', linestyle='--', label='Target Buffer Level')
plt.title('Buffer Tank Levels')
plt.xlabel('Time (minutes)')
plt.ylabel('Buffer Level (units)')
plt.legend()

plt.subplot(2, 1, 2)
plt.plot(infeed_rate_mpc, label='Infeed Rate (MPC)')
plt.plot(infeed_rate_pid, label='Infeed Rate (PID)')
plt.title('Infeed Rates')
plt.xlabel('Time (minutes)')
plt.ylabel('Infeed Rate (units/min)')
plt.legend()

plt.tight_layout()
plt.show()



