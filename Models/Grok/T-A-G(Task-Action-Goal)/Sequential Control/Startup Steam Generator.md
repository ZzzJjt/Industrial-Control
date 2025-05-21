import casadi as ca
import numpy as np
import matplotlib.pyplot as plt

# --- Process Model ---
def steam_generator_model():
    """
    Define non-linear ODE model for steam generator.
    States: T (temperature, °C), P (pressure, bar)
    Inputs: Q (feedwater flow, kg/s), F (fuel rate, kg/s)
    Returns: ODE function for integration
    """
    # States
    T = ca.MX.sym('T')  # Temperature (°C)
    P = ca.MX.sym('P')  # Pressure (bar)
    x = ca.vertcat(T, P)
    
    # Inputs
    Q = ca.MX.sym('Q')  # Feedwater flow (kg/s)
    F = ca.MX.sym('F')  # Fuel rate (kg/s)
    u = ca.vertcat(Q, F)
    
    # Parameters (simplified for demonstration)
    C_t = 1000.0  # Thermal capacitance (°C/(kg/s))
    C_p = 50.0    # Pressure capacitance (bar/(kg/s))
    k_f = 50.0    # Fuel-to-temperature gain (°C/(kg/s))
    k_q = 0.5     # Flow-to-pressure gain (bar/(kg/s))
    T_amb = 20.0  # Ambient temperature (°C)
    
    # Non-linear dynamics
    # dT/dt = (Fuel heat input - Heat loss + Feedwater effect) / Capacitance
    dT = (k_f * F - 0.1 * (T - T_amb) + 0.01 * Q * (T_amb - T)) / C_t
    # dP/dt = (Feedwater input - Steam output) / Capacitance
    dP = (k_q * Q - 0.05 * P) / C_p
    
    # ODE system
    xdot = ca.vertcat(dT, dP)
    
    return ca.Function('model', [x, u], [xdot], ['x', 'u'], ['xdot'])

# --- NMPC Setup ---
def setup_nmpc():
    """
    Configure NMPC optimization problem.
    Returns: NLP solver, bounds, and initial guess
    """
    # Model
    model = steam_generator_model()
    
    # Time settings
    T_step = 60.0  # Time step (seconds, 1 minute)
    N_pred = 20    # Prediction horizon (steps)
    N_ctrl = 10    # Control horizon (steps)
    
    # Variables
    x = ca.MX.sym('x', 2, N_pred + 1)  # States: [T, P] over horizon
    u = ca.MX.sym('u', 2, N_ctrl)      # Inputs: [Q, F] over control horizon
    t_final = ca.MX.sym('t_final')     # Final time (min)
    
    # Objective: Minimize startup time
    J = t_final
    
    # Constraints
    g = []
    
    # Initial state
    x0 = ca.vertcat(20.0, 1.0)  # Initial: T=20°C, P=1 bar
    g.append(x[:, 0] - x0)
    
    # Dynamics constraints (discretized using RK4)
    for k in range(N_pred):
        k1 = model(x=x[:, k], u=u[:, min(k, N_ctrl-1)])['xdot']
        k2 = model(x=x[:, k] + T_step/2*k1, u=u[:, min(k, N_ctrl-1)])['xdot']
        k3 = model(x=x[:, k] + T_step/2*k2, u=u[:, min(k, N_ctrl-1)])['xdot']
        k4 = model(x=x[:, k] + T_step*k3, u=u[:, min(k, N_ctrl-1)])['xdot']
        x_next = x[:, k] + T_step/6 * (k1 + 2*k2 + 2*k3 + k4)
        g.append(x[:, k+1] - x_next)
    
    # State constraints
    T_max = 520.0  # Max temperature (°C)
    P_min = 30.0   # Min pressure (bar)
    dT_max = 5.0 / 60.0  # Max temperature rate (°C/s)
    dP_max = 2.0 / 60.0  # Max pressure rate (bar/s)
    
    for k in range(N_pred + 1):
        g.append(x[0, k] - T_max)  # T <= 520°C
        g.append(P_min - x[1, k])  # P >= 30 bar
        if k < N_pred:
            g.append((x[0, k+1] - x[0, k]) / T_step - dT_max)  # dT/dt <= 5°C/min
            g.append(-(x[0, k+1] - x[0, k]) / T_step - dT_max) # -dT/dt <= 5°C/min
            g.append((x[1, k+1] - x[1, k]) / T_step - dP_max)  # dP/dt <= 2 bar/min
            g.append(-(x[1, k+1] - x[1, k]) / T_step - dP_max) # -dP/dt <= 2 bar/min
    
    # Input constraints
    Q_min, Q_max = 0.0, 50.0  # Feedwater flow bounds (kg/s)
    F_min, F_max = 0.0, 10.0  # Fuel rate bounds (kg/s)
    
    for k in range(N_ctrl):
        g.append(u[0, k] - Q_max)
        g.append(Q_min - u[0, k])
        g.append(u[1, k] - F_max)
        g.append(F_min - u[1, k])
    
    # Terminal constraint (reach operating condition)
    T_target = 500.0  # Target temperature (°C)
    P_target = 100.0  # Target pressure (bar)
    g.append(x[0, N_pred] - T_target)
    g.append(x[1, N_pred] - P_target)
    
    # NLP problem
    nlp = {'x': ca.vertcat(ca.vec(x), ca.vec(u), t_final), 'f': J, 'g': ca.vertcat(*g)}
    
    # Solver options
    opts = {'ipopt.print_level': 0, 'print_time': 0, 'ipopt.tol': 1e-6}
    solver = ca.nlpsol('solver', 'ipopt', nlp, opts)
    
    # Bounds
    lbx = [-ca.inf] * (2 * (N_pred + 1) + 2 * N_ctrl) + [0.0]  # t_final >= 0
    ubx = [ca.inf] * (2 * (N_pred + 1) + 2 * N_ctrl) + [ca.inf]
    lbg = [-ca.inf] * len(g)
    ubg = [0.0] * len(g)  # Inequality constraints <= 0
    
    return solver, lbx, ubx, lbg, ubg, x, u, t_final, N_pred, N_ctrl

# --- NMPC Optimization Loop ---
def nmpc_control_loop():
    """
    Execute NMPC control loop for steam generator startup.
    Returns: Time series of states and inputs
    """
    solver, lbx, ubx, lbg, ubg, x, u, t_final, N_pred, N_ctrl = setup_nmpc()
    
    # Simulation settings
    T_sim = 3600  # Simulation time (seconds, ~60 min)
    T_step = 60.0  # Control step (seconds, 1 min)
    N_sim = int(T_sim / T_step)  # Number of simulation steps
    
    # Initialize
    x0 = np.array([20.0, 1.0])  # Initial state: T=20°C, P=1 bar
    u0 = np.zeros((2, N_ctrl))  # Initial inputs: Q=0, F=0
    t_final0 = 60.0  # Initial guess for final time (min)
    
    # Storage for results
    t_history = [0.0]
    x_history = [x0]
    u_history = []
    
    # Current state
    x_current = x0
    
    # Control loop
    for k in range(N_sim):
        # Prepare initial guess
        x_guess = np.tile(x_current, (N_pred + 1, 1)).T
        x_guess[0, :] = np.linspace(x_current[0], 500.0, N_pred + 1)  # Linear temp profile
        x_guess[1, :] = np.linspace(x_current[1], 100.0, N_pred + 1)  # Linear pressure profile
        u_guess = np.tile(u0[:, 0], (N_ctrl, 1)).T
        init_guess = ca.vertcat(ca.vec(x_guess), ca.vec(u_guess), t_final0)
        
        # Update bounds for initial state
        lbg[0:2] = x_current
        ubg[0:2] = x_current
        
        # Solve NLP
        sol = solver(x0=init_guess, lbx=lbx, ubx=ubx, lbg=lbg, ubg=ubg)
        sol_x = sol['x'].full().flatten()
        
        # Extract solution
        x_opt = sol_x[:2 * (N_pred + 1)].reshape((2, N_pred + 1))
        u_opt = sol_x[2 * (N_pred + 1):2 * (N_pred + 1) + 2 * N_ctrl].reshape((2, N_ctrl))
        t_final_opt = sol_x[-1]
        
        # Apply first control action
        u_apply = u_opt[:, 0]
        u_history.append(u_apply)
        
        # Simulate system (RK4 integration)
        model = steam_generator_model()
        k1 = model(x=x_current, u=u_apply)['xdot'].full().flatten()
        k2 = model(x=x_current + T_step/2*k1, u=u_apply)['xdot'].full().flatten()
        k3 = model(x=x_current + T_step/2*k2, u=u_apply)['xdot'].full().flatten()
        k4 = model(x=x_current + T_step*k3, u=u_apply)['xdot'].full().flatten()
        x_current = x_current + T_step/6 * (k1 + 2*k2 + 2*k3 + k4)
        
        # Store results
        t_history.append(t_history[-1] + T_step / 60.0)  # Time in minutes
        x_history.append(x_current)
        
        # Update initial guess for next iteration
        u0 = np.hstack((u_opt[:, 1:], u_opt[:, -1:]))
        t_final0 = t_final_opt
        
        # Check if target reached
        if abs(x_current[0] - 500.0) < 1.0 and abs(x_current[1] - 100.0) < 1.0:
            break
    
    return np.array(t_history), np.array(x_history), np.array(u_history)

# --- Main Execution ---
if __name__ == "__main__":
    # Run NMPC
    t, x, u = nmpc_control_loop()
    
    # Plot results
    plt.figure(figsize=(12, 8))
    
    # States
    plt.subplot(2, 1, 1)
    plt.plot(t, x[:, 0], label='Temperature (°C)')
    plt.plot(t, x[:, 1], label='Pressure (bar)')
    plt.axhline(y=520, color='r', linestyle='--', label='T_max = 520°C')
    plt.axhline(y=30, color='g', linestyle='--', label='P_min = 30 bar')
    plt.xlabel('Time (min)')
    plt.ylabel('States')
    plt.legend()
    plt.grid(True)
    
    # Inputs
    plt.subplot(2, 1, 2)
    plt.step(t[:-1], u[:, 0], label='Feedwater Flow (kg/s)')
    plt.step(t[:-1], u[:, 1], label='Fuel Rate (kg/s)')
    plt.axhline(y=50, color='r', linestyle='--', label='Q_max = 50 kg/s')
    plt.axhline(y=10, color='g', linestyle='--', label='F_max = 10 kg/s')
    plt.xlabel('Time (min)')
    plt.ylabel('Inputs')
    plt.legend()
    plt.grid(True)
    
    plt.tight_layout()
    plt.savefig('nmpc_results.png')
    plt.show()
    
    # Print startup time
    print(f"Startup time: {t[-1]:.2f} minutes")
