import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt

# Reactor Model (Nonlinear CSTR with uncertainty)
class Reactor:
    def __init__(self, dt=1.0):
        self.dt = dt  # Time step (s)
        self.T = 20.0  # Temperature (°C)
        self.P = 2.0   # Pressure (bar)
        self.r = 0.05  # Reaction rate (mol/L/s)
        self.K_T = 0.5  # Temperature gain (°C/%)
        self.K_P = 0.1  # Pressure gain (bar/%)
        self.K_r = 0.002  # Reaction rate gain (mol/L/s/%)
        self.tau = 50.0  # Time constant (s)
        self.noise_std = 0.1  # Pressure measurement noise (bar)

    def step(self, u_T, u_P, u_r, P_disturb=0.0):
        """Simulate reactor dynamics with nonlinear effects and disturbance"""
        # Add disturbance to pressure
        P_eff = self.P + P_disturb
        # Nonlinear dynamics: temperature and pressure affect reaction rate
        r_eff = self.r * (1 + 0.01 * (self.T - 75) + 0.02 * (P_eff - 5))
        # Dynamics: dT/dt = -T/tau + K_T*u_T/tau
        dT_dt = (-self.T + self.K_T * u_T) / self.tau
        # dP/dt = -P/tau + K_P*u_P/tau + r_eff
        dP_dt = (-self.P + self.K_P * u_P + r_eff) / self.tau
        # dr/dt = -r/tau + K_r*u_r/tau
        dr_dt = (-self.r + self.K_r * u_r) / self.tau
        
        self.T += dT_dt * self.dt
        self.P += dP_dt * self.dt
        self.r += dr_dt * self.dt
        
        # Add measurement noise to pressure
        P_measured = self.P + np.random.normal(0, self.noise_std)
        return self.T, P_measured, self.r

# PID Controller (for comparison)
class PID:
    def __init__(self, Kp=2.0, Ki=0.01, Kd=0.5, dt=1.0):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.dt = dt
        self.integral = 0.0
        self.prev_error = 0.0

    def compute(self, setpoint, pv):
        """Compute PID control signal"""
        error = setpoint - pv
        self.integral += error * self.dt
        derivative = (error - self.prev_error) / self.dt
        u = self.Kp * error + self.Ki * self.integral + self.Kd * derivative
        self.prev_error = error
        return np.clip(u, 0, 100)

# Fuzzy Logic Controller
class FuzzyController:
    def __init__(self):
        # Define universes of discourse
        T_error = ctrl.Antecedent(np.arange(-10, 10.1, 0.1), 'T_error')
        P_error = ctrl.Antecedent(np.arange(-2, 2.1, 0.01), 'P_error')
        r = ctrl.Antecedent(np.arange(0, 0.21, 0.001), 'reaction_rate')
        u_T = ctrl.Consequent(np.arange(0, 100.1, 0.1), 'u_T')
        u_P = ctrl.Consequent(np.arange(0, 100.1, 0.1), 'u_P')
        u_r = ctrl.Consequent(np.arange(0, 100.1, 0.1), 'u_r')
        
        # Membership functions
        T_error['Low'] = fuzz.trimf(T_error.universe, [-10, -5, 0])
        T_error['Medium'] = fuzz.trimf(T_error.universe, [-2, 0, 2])
        T_error['High'] = fuzz.trimf(T_error.universe, [0, 5, 10])
        
        P_error['Low'] = fuzz.trimf(P_error.universe, [-2, -1, 0])
        P_error['Medium'] = fuzz.trimf(P_error.universe, [-0.5, 0, 0.5])
        P_error['High'] = fuzz.trimf(P_error.universe, [0, 1, 2])
        
        r['Low'] = fuzz.trimf(r.universe, [0, 0, 0.05])
        r['Medium'] = fuzz.trimf(r.universe, [0.05, 0.1, 0.15])
        r['High'] = fuzz.trimf(r.universe, [0.15, 0.2, 0.2])
        
        u_T['Decrease'] = fuzz.trimf(u_T.universe, [0, 0, 50])
        u_T['Maintain'] = fuzz.trimf(u_T.universe, [25, 50, 75])
        u_T['Increase'] = fuzz.trimf(u_T.universe, [50, 100, 100])
        
        u_P['Decrease'] = fuzz.trimf(u_P.universe, [0, 0, 50])
        u_P['Maintain'] = fuzz.trimf(u_P.universe, [25, 50, 75])
        u_P['Increase'] = fuzz.trimf(u_P.universe, [50, 100, 100])
        
        u_r['Decrease'] = fuzz.trimf(u_r.universe, [0, 0, 50])
        u_r['Maintain'] = fuzz.trimf(u_r.universe, [25, 50, 75])
        u_r['Increase'] = fuzz.trimf(u_r.universe, [50, 100, 100])
        
        # Fuzzy rules (example subset of 27 rules for brevity)
        rules = [
            ctrl.Rule(T_error['Low'] & P_error['Medium'] & r['Medium'], 
                     (u_T['Increase'], u_P['Maintain'], u_r['Maintain'])),
            ctrl.Rule(T_error['High'] & P_error['Medium'] & r['Medium'], 
                     (u_T['Decrease'], u_P['Maintain'], u_r['Maintain'])),
            ctrl.Rule(T_error['Medium'] & P_error['High'] & r['Medium'], 
                     (u_T['Maintain'], u_P['Decrease'], u_r['Maintain'])),
            ctrl.Rule(T_error['Medium'] & P_error['Low'] & r['Medium'], 
                     (u_T['Maintain'], u_P['Increase'], u_r['Maintain'])),
            ctrl.Rule(T_error['Medium'] & P_error['Medium'] & r['High'], 
                     (u_T['Maintain'], u_P['Maintain'], u_r['Decrease'])),
            ctrl.Rule(T_error['Medium'] & P_error['Medium'] & r['Low'], 
                     (u_T['Maintain'], u_P['Maintain'], u_r['Increase']))
        ]
        
        # Fuzzy control system
        self.system = ctrl.ControlSystem(rules)
        self.sim = ctrl.ControlSystemSimulator(self.system)
    
    def compute(self, T_error, P_error, r):
        """Compute fuzzy control signals"""
        self.sim.input['T_error'] = T_error
        self.sim.input['P_error'] = P_error
        self.sim.input['reaction_rate'] = r
        try:
            self.sim.compute()
            u_T = self.sim.output['u_T']
            u_P = self.sim.output['u_P']
            u_r = self.sim.output['u_r']
        except:
            # Fallback to maintain if computation fails
            u_T, u_P, u_r = 50.0, 50.0, 50.0
        return np.clip(u_T, 0, 100), np.clip(u_P, 0, 100), np.clip(u_r, 0, 100)

# Simulate Control Performance
def simulate(reactor, fuzzy, pid_T, pid_P, pid_r, t_sim=2000, dt=1.0):
    """Simulate fuzzy and PID control with pressure disturbance"""
    t = np.arange(0, t_sim, dt)
    T_fuzzy, P_fuzzy, r_fuzzy, uT_fuzzy, uP_fuzzy, ur_fuzzy = [], [], [], [], [], []
    T_pid, P_pid, r_pid, uT_pid, uP_pid, ur_pid = [], [], [], [], [], []
    SP_T, SP_P, SP_r = 75.0, 5.0, 0.1  # Setpoints
    
    # Reset reactor and PID controllers
    reactor.T, reactor.P, reactor.r = 20.0, 2.0, 0.05
    for pid in [pid_T, pid_P, pid_r]:
        pid.integral = 0.0
        pid.prev_error = 0.0
    
    # Fuzzy simulation
    for i in range(len(t)):
        T_error = SP_T - reactor.T
        P_error = SP_P - reactor.P
        P_disturb = 2.0 if 1000 <= t[i] < 1100 else 0.0  # Pressure spike at t=1000–1100s
        u_T, u_P, u_r = fuzzy.compute(T_error, P_error, reactor.r)
        T, P, r = reactor.step(u_T, u_P, u_r, P_disturb)
        T_fuzzy.append(T)
        P_fuzzy.append(P)
        r_fuzzy.append(r)
        uT_fuzzy.append(u_T)
        uP_fuzzy.append(u_P)
        ur_fuzzy.append(u_r)
    
    # Reset reactor for PID simulation
    reactor.T, reactor.P, reactor.r = 20.0, 2.0, 0.05
    
    # PID simulation
    for i in range(len(t)):
        P_disturb = 2.0 if 1000 <= t[i] < 1100 else 0.0
        u_T = pid_T.compute(SP_T, reactor.T)
        u_P = pid_P.compute(SP_P, reactor.P)
        u_r = pid_r.compute(SP_r, reactor.r)
        T, P, r = reactor.step(u_T, u_P, u_r, P_disturb)
        T_pid.append(T)
        P_pid.append(P)
        r_pid.append(r)
        uT_pid.append(u_T)
        uP_pid.append(u_P)
        ur_pid.append(u_r)
    
    return t, T_fuzzy, P_fuzzy, r_fuzzy, uT_fuzzy, uP_fuzzy, ur_fuzzy, \
           T_pid, P_pid, r_pid, uT_pid, uP_pid, ur_pid, SP_T, SP_P, SP_r

# Plot Results
def plot_results(t, T_fuzzy, P_fuzzy, r_fuzzy, uT_fuzzy, uP_fuzzy, ur_fuzzy,
                 T_pid, P_pid, r_pid, uT_pid, uP_pid, ur_pid, SP_T, SP_P, SP_r):
    """Plot fuzzy vs PID performance"""
    plt.figure(figsize=(12, 12))
    
    # Temperature
    plt.subplot(4, 1, 1)
    plt.plot(t, T_fuzzy, label='Fuzzy')
    plt.plot(t, T_pid, label='PID')
    plt.axhline(SP_T, color='r', linestyle='--', label='Setpoint')
    plt.ylabel('Temperature (°C)')
    plt.title('Reactor Temperature Control')
    plt.legend()
    
    # Pressure
    plt.subplot(4, 1, 2)
    plt.plot(t, P_fuzzy, label='Fuzzy')
    plt.plot(t, P_pid, label='PID')
    plt.axhline(SP_P, color='r', linestyle='--', label='Setpoint')
    plt.ylabel('Pressure (bar)')
    plt.title('Reactor Pressure Control')
    plt.legend()
    
    # Reaction Rate
    plt.subplot(4, 1, 3)
    plt.plot(t, r_fuzzy, label='Fuzzy')
    plt.plot(t, r_pid, label='PID')
    plt.axhline(SP_r, color='r', linestyle='--', label='Setpoint')
    plt.ylabel('Reaction Rate (mol/L/s)')
    plt.title('Reaction Rate Control')
    plt.legend()
    
    # Control Signals
    plt.subplot(4, 1, 4)
    plt.plot(t, uT_fuzzy, label='Fuzzy u_T')
    plt.plot(t, uT_pid, label='PID u_T')
    plt.plot(t, uP_fuzzy, label='Fuzzy u_P')
    plt.plot(t, uP_pid, label='PID u_P')
    plt.plot(t, ur_fuzzy, label='Fuzzy u_r')
    plt.plot(t, ur_pid, label='PID u_r')
    plt.ylabel('Control Signals (%)')
    plt.xlabel('Time (s)')
    plt.title('Control Signals')
    plt.legend()
    
    plt.tight_layout()
    plt.savefig('reactor_control.png')
    plt.show()

# Main Execution
if __name__ == "__main__":
    # Initialize components
    reactor = Reactor(dt=1.0)
    fuzzy = FuzzyController()
    pid_T = PID(Kp=2.0, Ki=0.01, Kd=0.5, dt=1.0)  # Temperature PID
    pid_P = PID(Kp=1.0, Ki=0.005, Kd=0.2, dt=1.0)  # Pressure PID
    pid_r = PID(Kp=50.0, Ki=0.1, Kd=5.0, dt=1.0)   # Reaction rate PID
    
    # Simulate control performance
    t, T_fuzzy, P_fuzzy, r_fuzzy, uT_fuzzy, uP_fuzzy, ur_fuzzy, \
    T_pid, P_pid, r_pid, uT_pid, uP_pid, ur_pid, SP_T, SP_P, SP_r = \
        simulate(reactor, fuzzy, pid_T, pid_P, pid_r, t_sim=2000)
    
    # Plot results
    plot_results(t, T_fuzzy, P_fuzzy, r_fuzzy, uT_fuzzy, uP_fuzzy, ur_fuzzy,
                 T_pid, P_pid, r_pid, uT_pid, uP_pid, ur_pid, SP_T, SP_P, SP_r)
    
    # Calculate performance metrics
    mse_T_fuzzy = np.mean((np.array(T_fuzzy) - SP_T) ** 2)
    mse_T_pid = np.mean((np.array(T_pid) - SP_T) ** 2)
    mse_P_fuzzy = np.mean((np.array(P_fuzzy) - SP_P) ** 2)
    mse_P_pid = np.mean((np.array(P_pid) - SP_P) ** 2)
    mse_r_fuzzy = np.mean((np.array(r_fuzzy) - SP_r) ** 2)
    mse_r_pid = np.mean((np.array(r_pid) - SP_r) ** 2)
    
    print(f"Fuzzy Temperature MSE: {mse_T_fuzzy:.2f} °C²")
    print(f"PID Temperature MSE: {mse_T_pid:.2f} °C²")
    print(f"Fuzzy Pressure MSE: {mse_P_fuzzy:.2f} bar²")
    print(f"PID Pressure MSE: {mse_P_pid:.2f} bar²")
    print(f"Fuzzy Reaction Rate MSE: {mse_r_fuzzy:.6f} (mol/L/s)²")
    print(f"PID Reaction Rate MSE: {mse_r_pid:.6f} (mol/L/s)²")
