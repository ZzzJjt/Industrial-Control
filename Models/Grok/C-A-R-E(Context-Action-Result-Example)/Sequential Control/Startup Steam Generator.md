from gekko import GEKKO
import numpy as np

def create_nmpc_controller():
    """
    Creates and configures an NMPC controller for steam generator startup.
    Returns a GEKKO model with prediction, constraints, and optimization setup.
    """
    # Initialize GEKKO model (remote=False for local computation)
    m = GEKKO(remote=False)

    # Time settings
    m.time = np.linspace(0, 60, 61)  # 60-second prediction horizon, 1-second steps
    N = len(m.time)  # Prediction horizon length

    # Parameters (simplified for model)
    C_b = 1000.0  # Boiler thermal capacitance (J/°C)
    C_s = 500.0   # Steam thermal capacitance (J/°C)
    Q_f = 2e6     # Fuel heating value (J/kg)
    h_b = 1000.0  # Heat transfer coefficient (W/°C)
    V_b = 10.0    # Boiler volume (m³)
    m_w = 2.0     # Water mass flow (kg/s)
    h_v = 2e6     # Vaporization enthalpy (J/kg)
    c_p = 2000.0  # Specific heat of steam (J/kg·°C)

    # States (initial conditions)
    T_b = m.Var(value=100.0, lb=50.0, ub=550.0)  # Boiler temp (°C)
    P_b = m.Var(value=10.0, lb=30.0, ub=100.0)  # Boiler pressure (bar)
    T_s = m.Var(value=100.0, lb=50.0, ub=520.0)  # Steam temp (°C)

    # Control inputs
    u_f = m.MV(value=0.0, lb=0.0, ub=10.0)  # Fuel flow (kg/s)
    u_s = m.MV(value=0.0, lb=0.0, ub=5.0)   # Steam flow (kg/s)

    # Enable control inputs with rate limits
    u_f.STATUS = 1
    u_f.DCOST = 0.5  # Penalize control changes
    u_s.STATUS = 1
    u_s.DCOST = 0.2  # Penalize control changes

    # Non-linear model equations
    m.Equation(T_b.dt() == (1/C_b) * (u_f * Q_f - h_b * (T_b - T_s)))
    m.Equation(P_b.dt() == (1/V_b) * (m_w * h_v - u_s))
    m.Equation(T_s.dt() == (1/C_s) * (h_b * (T_b - T_s) - u_s * c_p * T_s))

    # Objective function
    w1, w2, w3, w4 = 10.0, 5.0, 1.0, 1.0  # Weights
    m.Obj(w1 * (P_b - 90.0)**2)  # Minimize pressure deviation
    m.Obj(w2 * (T_s - 500.0)**2)  # Minimize steam temp deviation
    m.Obj(w3 * u_f.dt()**2)       # Minimize fuel flow changes
    m.Obj(w4 * u_s.dt()**2)       # Minimize steam flow changes

    # Solver settings
    m.options.IMODE = 6  # MPC mode
    m.options.SOLVER = 3 # IPOPT solver
    m.options.MAX_ITER = 500

    return m, T_b, P_b, T_s, u_f, u_s

def simulate_startup():
    """
    Simulates the steam generator startup using NMPC.
    Returns time, states, and control inputs for analysis.
    """
    # Create NMPC model
    m, T_b, P_b, T_s, u_f, u_s = create_nmpc_controller()

    # Simulation settings
    sim_time = 1800  # 30 minutes (seconds)
    time_steps = int(sim_time / 1.0)  # 1-second control interval
    time = np.linspace(0, sim_time, time_steps + 1)

    # Storage for results
    T_b_traj = np.zeros(time_steps + 1)
    P_b_traj = np.zeros(time_steps + 1)
    T_s_traj = np.zeros(time_steps + 1)
    u_f_traj = np.zeros(time_steps + 1)
    u_s_traj = np.zeros(time_steps + 1)

    # Initial conditions
    T_b_traj[0] = T_b.value[0]
    P_b_traj[0] = P_b.value[0]
    T_s_traj[0] = T_s.value[0]
    u_f_traj[0] = u_f.value[0]
    u_s_traj[0] = u_s.value[0]

    # Simulation loop
    for i in range(time_steps):
        try:
            m.solve(disp=False)  # Solve NMPC problem
            # Store results
            T_b_traj[i + 1] = T_b.value[0]
            P_b_traj[i + 1] = P_b.value[0]
            T_s_traj[i + 1] = T_s.value[0]
            u_f_traj[i + 1] = u_f.value[0]
            u_s_traj[i + 1] = u_s.value[0]
            # Update initial conditions for next step
            T_b.value = T_b.value[0]
            P_b.value = P_b.value[0]
            T_s.value = T_s.value[0]
        except Exception as e:
            print(f"Solver failed at step {i}: {e}")
            break

    return time, T_b_traj, P_b_traj, T_s_traj, u_f_traj, u_s_traj

if __name__ == "__main__":
    # Run simulation
    time, T_b_traj, P_b_traj, T_s_traj, u_f_traj, u_s_traj = simulate_startup()

    # Optional: Plot results (requires matplotlib)
    try:
        import matplotlib.pyplot as plt
        plt.figure(figsize=(12, 8))
        plt.subplot(3, 1, 1)
        plt.plot(time, T_b_traj, label='Boiler Temp (°C)')
        plt.plot(time, T_s_traj, label='Steam Temp (°C)')
        plt.axhline(550, color='r', linestyle='--', label='T_b Max')
        plt.axhline(520, color='g', linestyle='--', label='T_s Max')
        plt.legend()
        plt.grid()

        plt.subplot(3, 1, 2)
        plt.plot(time, P_b_traj, label='Boiler Pressure (bar)')
        plt.axhline(90, color='b', linestyle='--', label='Target')
        plt.axhline(30, color='r', linestyle='--', label='Min')
        plt.legend()
        plt.grid()

        plt.subplot(3, 1, 3)
        plt.plot(time, u_f_traj, label='Fuel Flow (kg/s)')
        plt.plot(time, u_s_traj, label='Steam Flow (kg/s)')
        plt.legend()
        plt.grid()

        plt.savefig('steam_generator_startup.png')
        plt.close()
    except ImportError:
        print("Matplotlib not installed. Skipping plot.")
