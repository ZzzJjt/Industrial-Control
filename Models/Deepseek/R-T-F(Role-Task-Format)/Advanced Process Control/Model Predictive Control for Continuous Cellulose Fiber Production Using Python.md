import numpy as np
import casadi.tools as ct
from do_mpc.controller import MPC
from do_mpc.model import Model
from do_mpc.simulator import Simulator
from do_mpc.estimator import StateFeedback
import time

# Step 1: Define the Model
model_type = 'continuous'  # Either 'discrete' or 'continuous'
model = Model(model_type)

# States
T = model.set_variable('_x', 'T')  # Temperature in reactor (not used directly but part of state)
V_buffer = model.set_variable('_x', 'V_buffer')  # Volume in buffer tank [m^3]

# Inputs
u_feed = model.set_variable('_u', 'u_feed')  # Feed rate [m^3/hour]

# Disturbances
d_demand = model.set_variable('_z', 'd_demand')  # Demand disturbance [m^3/hour]

# Parameters
k_prod = model.set_variable('_p', 'k_prod')  # Production rate constant [m^3/hour]
tau_delay = 2  # Time delay in hours

# Differential Equations
model.set_rhs('T', 0)  # Placeholder for temperature dynamics (not used directly)
model.set_rhs('V_buffer', u_feed * tau_delay - d_demand)

# Measurement Equation
model.set_meas('meas_V_buffer', V_buffer)

# Build the model
model.setup()

# Step 2: Setup the MPC Controller
mpc = MPC(model)

# Objective function
setup_mpc = {
    'n_horizon': 20,
    't_step': 1,  # Sampling time in hours
    'state_discretization': 'collocation',
    'collocation_type': 'radau',
    'collocation_deg': 3,
    'store_full_solution': True,
}

mpc.set_param(**setup_mpc)

# Cost function
mpc.set_objective(mterm=1e-5*V_buffer**2, lterm=(V_buffer - 800)**2 + 1e-4*u_feed**2)

# Bounds
mpc.bounds['lower', '_x', 'V_buffer'] = 0
mpc.bounds['upper', '_x', 'V_buffer'] = 1000
mpc.bounds['lower', '_u', 'u_feed'] = 0
mpc.bounds['upper', '_u', 'u_feed'] = 200
mpc.bounds['lower', '_z', 'd_demand'] = 0
mpc.bounds['upper', '_z', 'd_demand'] = 100

# Terminal bounds
mpc.bounds['soft', '_x', 'V_buffer', ub=900, lb=700]
mpc.set_rterm(V_buffer=1e-2, u_feed=1e-4)

# Set initial guess
mpc.set_initial_guess()

# Setup optimizer
mpc.setup()

# Step 3: Setup the Estimator
estimator = StateFeedback(model)

# Step 4: Setup the Simulator
simulator = Simulator(model)

# Initial conditions
x0 = np.array([300, 500])  # Initial temperature and volume in buffer tank
simulator.x0 = x0
estimator.x0 = x0

# Initial parameters
p0 = {'k_prod': 10}  # Example parameter value
simulator.set_p_fun(lambda t_now: p0)
estimator.set_p_fun(lambda t_now: p0)

# Step 5: Run Simulation
n_steps = 60  # Total simulation steps (1 hour each)
log_tank_levels = []
log_feed_rates = []
log_demands = []

for _ in range(n_steps):
    # Get current measurement
    y_next = simulator.make_step(u0)
    
    # Generate new demand disturbance
    d_demand_val = 50 + 20 * np.sin(time.time())  # Fluctuating demand
    
    # Make MPC step
    u_MPC = mpc.make_step(y_next, z_next=np.array([d_demand_val]))
    
    # Store results
    log_tank_levels.append(simulator.data['_x'][-1, 1])
    log_feed_rates.append(u_MPC[0])
    log_demands.append(d_demand_val)
    
    # Update last input
    u0 = u_MPC

# Print simulation results
print("Simulation Results:")
print(f"{'Time [hours]'}\t{'Tank Level [m^3]'}\t{'Feed Rate [m^3/hour]'}\t{'Demand [m^3/hour]'}")
for t in range(n_steps):
    print(f"{t}\t\t{log_tank_levels[t]:.2f}\t\t{log_feed_rates[t]:.2f}\t\t{log_demands[t]:.2f}")
