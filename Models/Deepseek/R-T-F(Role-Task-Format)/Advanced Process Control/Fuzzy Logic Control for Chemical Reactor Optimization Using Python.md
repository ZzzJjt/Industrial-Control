import numpy as np
import matplotlib.pyplot as plt
from skfuzzy import control as ctrl
from skfuzzy import membership
from scipy.integrate import solve_ivp

# Step 1: Define Fuzzy Sets and Rules

# Define universe variables
temperature = ctrl.Antecedent(np.arange(20, 80, 1), 'temperature')
pressure = ctrl.Antecedent(np.arange(1, 10, 0.1), 'pressure')
reaction_rate = ctrl.Consequent(np.arange(-10, 10, 0.1), 'reaction_rate')

# Define membership functions for temperature
temperature['low'] = membership.trimf(temperature.universe, [20, 20, 40])
temperature['medium'] = membership.trimf(temperature.universe, [30, 50, 70])
temperature['high'] = membership.trimf(temperature.universe, [60, 80, 80])

# Define membership functions for pressure
pressure['low'] = membership.trimf(pressure.universe, [1, 1, 5])
pressure['medium'] = membership.trimf(pressure.universe, [3, 5, 7])
pressure['high'] = membership.trimf(pressure.universe, [5, 9, 9])

# Define membership functions for reaction rate
reaction_rate['negative'] = membership.trimf(reaction_rate.universe, [-10, -10, 0])
reaction_rate['zero'] = membership.trimf(reaction_rate.universe, [-5, 0, 5])
reaction_rate['positive'] = membership.trimf(reaction_rate.universe, [0, 10, 10])

# Define fuzzy rules
rule1 = ctrl.Rule(temperature['low'] & pressure['low'], reaction_rate['positive'])
rule2 = ctrl.Rule(temperature['low'] & pressure['medium'], reaction_rate['positive'])
rule3 = ctrl.Rule(temperature['low'] & pressure['high'], reaction_rate['zero'])
rule4 = ctrl.Rule(temperature['medium'] & pressure['low'], reaction_rate['positive'])
rule5 = ctrl.Rule(temperature['medium'] & pressure['medium'], reaction_rate['zero'])
rule6 = ctrl.Rule(temperature['medium'] & pressure['high'], reaction_rate['negative'])
rule7 = ctrl.Rule(temperature['high'] & pressure['low'], reaction_rate['zero'])
rule8 = ctrl.Rule(temperature['high'] & pressure['medium'], reaction_rate['negative'])
rule9 = ctrl.Rule(temperature['high'] & pressure['high'], reaction_rate['negative'])

# Create control system
reactions_ctrl = ctrl.ControlSystem([rule1, rule2, rule3, rule4, rule5, rule6, rule7, rule8, rule9])
reactions = ctrl.ControlSystemSimulation(reactions_ctrl)

# Step 2: Implement Fuzzy Inference System

def fuzzy_controller(temp, pres):
    reactions.input['temperature'] = temp
    reactions.input['pressure'] = pres
    reactions.compute()
    return reactions.output['reaction_rate']

# Step 3: Simulate Reactor Behavior Under Various Scenarios

# Define reactor dynamics function
def reactor_dynamics(t, y, K_temp, K_pres, noise_std=0.1):
    temp, pres = y
    setpoint_temp = 50 + 10 * np.sin(t / 10)  # Time-varying temperature setpoint
    setpoint_pres = 5 + 1 * np.cos(t / 5)     # Time-varying pressure setpoint
    
    # Fuzzy controller output
    fuzzy_output = fuzzy_controller(temp, pres)
    
    # PID controller parameters
    Kp_temp, Ki_temp, Kd_temp = 1.0, 0.1, 0.01
    Kp_pres, Ki_pres, Kd_pres = 1.0, 0.1, 0.01
    
    # PID controller outputs
    global prev_error_temp, integral_temp, prev_error_pres, integral_pres
    
    error_temp = setpoint_temp - temp
    integral_temp += error_temp * dt
    derivative_temp = (error_temp - prev_error_temp) / dt
    pid_output_temp = Kp_temp * error_temp + Ki_temp * integral_temp + Kd_temp * derivative_temp
    prev_error_temp = error_temp
    
    error_pres = setpoint_pres - pres
    integral_pres += error_pres * dt
    derivative_pres = (error_pres - prev_error_pres) / dt
    pid_output_pres = Kp_pres * error_pres + Ki_pres * integral_pres + Kd_pres * derivative_pres
    prev_error_pres = error_pres
    
    # Reactor dynamics equations
    dtemp_dt = K_temp * (fuzzy_output + pid_output_temp) + np.random.normal(0, noise_std)
    dpres_dt = K_pres * (fuzzy_output + pid_output_pres) + np.random.normal(0, noise_std)
    
    return [dtemp_dt, dpres_dt]

# Initial conditions
initial_conditions = [30, 3]  # Initial temperature and pressure
K_temp = 0.5
K_pres = 0.5
dt = 0.1
total_time = 100
num_steps = int(total_time / dt)
time_points = np.linspace(0, total_time, num_steps)

# Initialize PID controller variables
prev_error_temp = 0.0
integral_temp = 0.0
prev_error_pres = 0.0
integral_pres = 0.0

# Solve ODEs
solution = solve_ivp(fun=lambda t, y: reactor_dynamics(t, y, K_temp, K_pres),
                     t_span=[0, total_time],
                     y0=initial_conditions,
                     t_eval=time_points,
                     method='RK45',
                     rtol=1e-6,
                     atol=1e-6)

# Extract results
temperatures_fuzzy_pid = solution.y[0]
pressures_fuzzy_pid = solution.y[1]

# Reset PID controller variables for PID-only simulation
prev_error_temp = 0.0
integral_temp = 0.0
prev_error_pres = 0.0
integral_pres = 0.0

# Define reactor dynamics function for PID only
def reactor_dynamics_pid(t, y, K_temp, K_pres, noise_std=0.1):
    temp, pres = y
    setpoint_temp = 50 + 10 * np.sin(t / 10)  # Time-varying temperature setpoint
    setpoint_pres = 5 + 1 * np.cos(t / 5)     # Time-varying pressure setpoint
    
    # PID controller parameters
    Kp_temp, Ki_temp, Kd_temp = 1.0, 0.1, 0.01
    Kp_pres, Ki_pres, Kd_pres = 1.0, 0.1, 0.01
    
    # PID controller outputs
    global prev_error_temp, integral_temp, prev_error_pres, integral_pres
    
    error_temp = setpoint_temp - temp
    integral_temp += error_temp * dt
    derivative_temp = (error_temp - prev_error_temp) / dt
    pid_output_temp = Kp_temp * error_temp + Ki_temp * integral_temp + Kd_temp * derivative_temp
    prev_error_temp = error_temp
    
    error_pres = setpoint_pres - pres
    integral_pres += error_pres * dt
    derivative_pres = (error_pres - prev_error_pres) / dt
    pid_output_pres = Kp_pres * error_pres + Ki_pres * integral_pres + Kd_pres * derivative_pres
    prev_error_pres = error_pres
    
    # Reactor dynamics equations
    dtemp_dt = K_temp * pid_output_temp + np.random.normal(0, noise_std)
    dpres_dt = K_pres * pid_output_pres + np.random.normal(0, noise_std)
    
    return [dtemp_dt, dpres_dt]

# Solve ODEs for PID only
solution_pid = solve_ivp(fun=lambda t, y: reactor_dynamics_pid(t, y, K_temp, K_pres),
                         t_span=[0, total_time],
                         y0=initial_conditions,
                         t_eval=time_points,
                         method='RK45',
                         rtol=1e-6,
                         atol=1e-6)

# Extract results for PID only
temperatures_pid = solution_pid.y[0]
pressures_pid = solution_pid.y[1]

# Step 4: Visualization Comparing Fuzzy Control vs. PID Performance

plt.figure(figsize=(14, 10))

# Temperature Comparison
plt.subplot(2, 1, 1)
plt.plot(time_points, temperatures_fuzzy_pid, label='Temperature (Fuzzy + PID)', linewidth=2)
plt.plot(time_points, temperatures_pid, label='Temperature (PID Only)', linestyle='--', linewidth=2)
plt.plot(time_points, 50 + 10 * np.sin(time_points / 10), label='Setpoint', linestyle=':', color='black')
plt.xlabel('Time [s]')
plt.ylabel('Temperature [°C]')
plt.title('Temperature Control Comparison')
plt.legend()
plt.grid(True)

# Pressure Comparison
plt.subplot(2, 1, 2)
plt.plot(time_points, pressures_fuzzy_pid, label='Pressure (Fuzzy + PID)', linewidth=2)
plt.plot(time_points, pressures_pid, label='Pressure (PID Only)', linestyle='--', linewidth=2)
plt.plot(time_points, 5 + 1 * np.cos(time_points / 5), label='Setpoint', linestyle=':', color='black')
plt.xlabel('Time [s]')
plt.ylabel('Pressure [bar]')
plt.title('Pressure Control Comparison')
plt.legend()
plt.grid(True)

plt.tight_layout()
plt.show()
