import numpy as np
from scipy.integrate import solve_ivp
from casadi import *

# Define parameters
buffer_tank_capacity = 1000  # m³
feed_rate = 50 / 3.6  # Conversion from tons/hour to m³/second assuming density=1 ton/m³
time_delay_hours = 2  # Hours
time_delay_seconds = time_delay_hours * 3600  # Convert hours to seconds

def batch_process(t, y, u):
    """
    Simulate the two-stage batch process including the time delay.
    :param t: Current time.
    :param y: State vector [reactor_level, homogenizer_level, tank_level].
    :param u: Input vector [feed_rate_input].
    :return: Derivatives of state variables.
    """
    reactor_level, homogenizer_level, tank_level = y
    feed_rate_input = u[0]

    d_reactor_dt = feed_rate_input - 0 if t < time_delay_seconds else feed_rate_input - feed_rate
    d_homogenizer_dt = max(0, feed_rate - homogenizer_level)
    d_tank_dt = max(0, homogenizer_level - tank_level)

    return [d_reactor_dt, d_homogenizer_dt, d_tank_dt]

# Initial conditions
y0 = [0, 0, 0]  # Initial levels in m³

# Simulation time span
t_span = (0, 86400)  # One day in seconds

# Solve ODEs
sol = solve_ivp(batch_process, t_span, y0, args=([feed_rate],), dense_output=True)

# Define MPC parameters
N = 10  # Prediction horizon
dt = 3600  # Sample time in seconds (1 hour)

opti = Opti()  # Optimization problem

# State variables
X = opti.variable(N+1, 3)  # States: reactor level, homogenizer level, tank level

# Control variable
U = opti.variable(N, 1)  # Feed rate input

# Parameters
P = opti.parameter(1)  # Parameter for demand forecast

# Objective function
obj = 0
for k in range(N):
    obj += (X[k, 2] - P)**2 + .1*(U[k] - feed_rate)**2
opti.minimize(obj)

# Dynamic constraints
for k in range(N):
    f = vertcat(
        U[k] - X[k, 0],
        max(0, feed_rate - X[k, 1]),
        max(0, X[k, 1] - X[k, 2])
    )
    opti.subject_to(X[k+1, :] == X[k, :] + f*dt)

# Constraints
opti.subject_to(opti.bounded(0, X[:, 2], buffer_tank_capacity))  # Tank level limits
opti.subject_to(opti.bounded(0, U, feed_rate*2))  # Feed rate limits

# Solver options
opti.solver('ipopt')

# Function to get optimal control action
def mpc_controller(current_state, demand_forecast):
    opti.set_value(P, demand_forecast)
    opti.set_initial(X, np.tile(current_state, (N+1, 1)))
    sol = opti.solve()
    return sol.value(U)[0]

  # Example demand scenario (random fluctuation around mean demand)
np.random.seed(0)
demand_scenarios = np.random.normal(loc=feed_rate, scale=feed_rate*.1, size=(10, N))

# Simulate each scenario
results = []
for scenario in demand_scenarios:
    state = np.array(y0)
    result = []
    for i in range(N):
        control_action = mpc_controller(state, scenario[i])
        state_next = batch_process(i*dt, state, [control_action])[0]
        state = state_next
        result.append(state[2])  # Collect tank level
    results.append(result)

# Evaluate the results numerically without plotting
for idx, res in enumerate(results):
    avg_tank_level = np.mean(res)
    min_tank_level = min(res)
    print(f"Scenario {idx + 1}: Avg Tank Level={avg_tank_level:.2f}m³, Min Tank Level={min_tank_level:.2f}m³")
