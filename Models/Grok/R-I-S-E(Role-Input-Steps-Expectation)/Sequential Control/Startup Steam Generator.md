model.py
```python
import numpy as np
from casadi import SX, vertcat, integrator

def create_steam_model():
    """
    Creates a non-linear ODE model for the steam generator.
    States: P (pressure, bar), T (temperature, °C)
    Inputs: u_f (fuel flow, normalized 0–1), u_w (water feed, normalized 0–1)
    """
    # State variables
    P = SX.sym('P')  # Pressure (bar)
    T = SX.sym('T')  # Temperature (°C)

    # Control inputs
    u_f = SX.sym('u_f')  # Fuel flow (0–1)
    u_w = SX.sym('u_w')  # Water feed (0–1)

    # Parameters (simplified, typical values)
    V = 10.0  # Reactor volume (m³)
    h_v = 2000.0  # Latent heat of vaporization (kJ/kg)
    C_p = 4.2  # Specific heat capacity (kJ/kg·°C)
    Q_max = 1000.0  # Max heat input (kJ/s)
    m_w_max = 1.0  # Max water feed rate (kg/s)
    P_0 = 30.0  # Initial pressure (bar)
    T_0 = 300.0  # Initial temperature (°C)

    # Non-linear dynamics (simplified mass and energy balance)
    # dP/dt: Pressure change due to mass balance
    dP_dt = (m_w_max * u_w - 0.01 * P) / V  # Simplified mass flow
    # dT/dt: Temperature change due to energy balance
    dT_dt = (Q_max * u_f - h_v * m_w_max * u_w) / (V * C_p)

    # State and input vectors
    x = vertcat(P, T)
    u = vertcat(u_f, u_w)
    xdot = vertcat(dP_dt, dT_dt)

    # Create integrator for simulation
    opts = {'tf': 1.0}  # Time step = 1 second
    F = integrator('F', 'cvodes', {'x': x, 'p': u, 'ode': xdot}, opts)

    return F, x, u
```

controller.py
```python
from casadi import SX, vertcat, nlpsol, vcat
import numpy as np

class NMPCController:
    """
    Non-linear Model Predictive Controller for steam generator startup.
    Optimizes fuel and water inputs to reach nominal conditions (P=40 bar, T=500°C).
    """
    def __init__(self, model, N_p=20, N_c=5, dt=1.0):
        """
        Initialize NMPC with model, horizons, and time step.
        Args:
            model: CasADi integrator function
            N_p: Prediction horizon (steps)
            N_c: Control horizon (steps)
            dt: Time step (seconds)
        """
        self.F, self.x, self.u = model
        self.N_p = N_p
        self.N_c = N_c
        self.dt = dt
        self.nx = 2  # States: P, T
        self.nu = 2  # Inputs: u_f, u_w

        # Setpoints
        self.P_set = 40.0  # Nominal pressure (bar)
        self.T_set = 500.0  # Nominal temperature (°C)

        # Constraints
        self.P_min = 30.0  # Min pressure (bar)
        self.T_max = 520.0  # Max temperature (°C)
        self.u_min = 0.0   # Min input
        self.u_max = 1.0   # Max input

        # Build NMPC problem
        self._build_nmpc()

    def _build_nmpc(self):
        """Formulate the NMPC optimization problem."""
        # Decision variables: states and inputs over horizons
        X = SX.sym('X', self.nx, self.N_p + 1)  # States: P, T
        U = SX.sym('U', self.nu, self.N_c)      # Inputs: u_f, u_w
        P = SX.sym('P', self.nx)                # Initial state

        # Cost function: minimize deviation from setpoints and control effort
        J = 0
        Q = np.diag([10.0, 10.0])  # Weight on state errors (P, T)
        R = np.diag([0.1, 0.1])    # Weight on control effort
        for k in range(self.N_p):
            # State error
            state_error = vertcat(X[0, k] - self.P_set, X[1, k] - self.T_set)
            J += state_error.T @ Q @ state_error
            # Control effort (only within control horizon)
            if k < self.N_c:
                J += U[:, k].T @ R @ U[:, k]

        # Constraints
        g = []
        # Initial condition
        g.append(X[:, 0] - P)

        # Dynamic constraints
        for k in range(self.N_p):
            # Control input (hold beyond N_c)
            u_k = U[:, k] if k < self.N_c else U[:, self.N_c - 1]
            # Integrate state
            x_next = self.F(x0=X[:, k], p=u_k)['xf']
            g.append(X[:, k + 1] - x_next)

        # Flatten variables for solver
        opt_vars = vcat([X.reshape((-1, 1)), U.reshape((-1, 1))])
        opt_params = P
        g = vcat(g)

        # Bounds
        lbx = [-float('inf')] * (self.nx * (self.N_p + 1) + self.nu * self.N_c)
        ubx = [float('inf')] * (self.nx * (self.N_p + 1) + self.nu * self.N_c)
        # State constraints
        for k in range(self.N_p + 1):
            lbx[k * self.nx] = self.P_min  # P >= 30 bar
            ubx[k * self.nx + 1] = self.T_max  # T <= 520°C
        # Input constraints
        for k in range(self.N_c):
            lbx[self.nx * (self.N_p + 1) + k * self.nu : 
                self.nx * (self.N_p + 1) + (k + 1) * self.nu] = [self.u_min] * self.nu
            ubx[self.nx * (self.N_p + 1) + k * self.nu : 
                self.nx * (self.N_p + 1) + (k + 1) * self.nu] = [self.u_max] * self.nu

        # NLP problem
        nlp = {'x': opt_vars, 'p': opt_params, 'f': J, 'g': g}
        self.solver = nlpsol('solver', 'ipopt', nlp, {'ipopt.print_level': 0})

        # Store bounds
        self.lbx = lbx
        self.ubx = ubx
        self.lbg = [0.0] * g.size1()
        self.ubg = [0.0] * g.size1()

    def compute_control(self, x0):
        """
        Compute optimal control action given current state.
        Args:
            x0: Current state [P, T]
        Returns:
            u_opt: Optimal control [u_f, u_w]
        """
        # Initial guess
        x_guess = np.zeros(self.nx * (self.N_p + 1) + self.nu * self.N_c)

        # Solve NLP
        sol = self.solver(x=x_guess, p=x0, lbx=self.lbx, ubx=self.ubx, 
                         lbg=self.lbg, ubg=self.ubg)
        opt_vars = sol['x'].full().flatten()

        # Extract first control action
        u_opt = opt_vars[self.nx * (self.N_p + 1) : self.nx * (self.N_p + 1) + self.nu]
        return u_opt
```

main.py
```python
import numpy as np
import matplotlib.pyplot as plt
from model import create_steam_model
from controller import NMPCController

def simulate_startup():
    """
    Simulate steam generator startup using NMPC.
    Plots states (P, T) and controls (u_f, u_w) over time.
    """
    # Create model and controller
    model = create_steam_model()
    nmpc = NMPCController(model, N_p=20, N_c=5, dt=1.0)

    # Simulation parameters
    t_max = 600  # Simulation time (seconds)
    dt = 1.0     # Time step (seconds)
    N_sim = int(t_max / dt)
    
    # Initial state
    x0 = np.array([30.0, 300.0])  # P=30 bar, T=300°C
    x_history = [x0]
    u_history = []

    # Simulate
    x = x0
    for t in range(N_sim):
        # Compute control action
        u = nmpc.compute_control(x)
        u_history.append(u)

        # Simulate one step
        x_next = model[0](x0=x, p=u)['xf'].full().flatten()
        x = x_next
        x_history.append(x)

        # Check constraints
        if x[0] < nmpc.P_min or x[1] > nmpc.T_max:
            print(f"Constraint violation at t={t*dt}s: P={x[0]:.2f}, T={x[1]:.2f}")
            break

    # Plot results
    t = np.linspace(0, t_max, N_sim + 1)
    x_history = np.array(x_history)
    u_history = np.array(u_history)

    plt.figure(figsize=(10, 8))
    plt.subplot(2, 1, 1)
    plt.plot(t, x_history[:, 0], label='Pressure (bar)')
    plt.plot(t, x_history[:, 1], label='Temperature (°C)')
    plt.axhline(nmpc.P_set, color='k', linestyle='--', label='P setpoint')
    plt.axhline(nmpc.T_set, color='r', linestyle='--', label='T setpoint')
    plt.xlabel('Time (s)')
    plt.ylabel('States')
    plt.legend()
    plt.grid()

    plt.subplot(2, 1, 2)
    plt.plot(t[:-1], u_history[:, 0], label='Fuel flow (u_f)')
    plt.plot(t[:-1], u_history[:, 1], label='Water feed (u_w)')
    plt.xlabel('Time (s)')
    plt.ylabel('Controls')
    plt.legend()
    plt.grid()

    plt.tight_layout()
    plt.show()

if __name__ == "__main__":
    simulate_startup()
```
