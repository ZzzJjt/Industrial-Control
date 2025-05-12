import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt
import scipy.integrate

# Set random seed for reproducibility
np.random.seed(42)

# Reactor simulation model
class ReactorModel:
    def __init__(self):
        self.T0 = 300  # Initial temperature (K)
        self.P0 = 100  # Initial pressure (kPa)
        self.R0 = 0.1  # Initial reaction rate (mol/s)
        self.Cp = 4.18  # Heat capacity (J/gK)
        self.m = 1000   # Mass (g)
        self.UA = 50    # Heat transfer coefficient (W/K)
        self.T_amb = 298  # Ambient temperature (K)
        self.V = 1.0    # Reactor volume (m^3)
        self.R = 8.314  # Gas constant (J/mol·K)
        
    def dynamics(self, t, state, inputs):
        T, P, R = state
        Q, F = inputs  # Heat input (W), Flow rate (m^3/s)
        
        # Nonlinear dynamics
        dTdt = (Q - self.UA * (T - self.T_amb)) / (self.m * self.Cp) - 0.001 * (T - self.T0)**2
        dPdt = (self.R * T * R - P * F) / self.V
        dRdt = 0.1 * (T/350)**2 * (P/100) - 0.05 * R  # Reaction kinetics
        
        return [dTdt, dPdt, dRdt]
    
    def simulate(self, t_span, inputs, state_init):
        sol = scipy.integrate.solve_ivp(
            fun=lambda t, state: self.dynamics(t, state, inputs(t)),
            t_span=(t_span[0], t_span[-1]),
            y0=state_init,
            t_eval=t_span
        )
        return sol.y

# PID Controller
class PIDController:
    def __init__(self, Kp=10.0, Ki=0.1, Kd=0.05):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.prev_errors = [0, 0]  # For temperature and pressure
        self.integrals = [0, 0]
        
    def control(self, setpoints, measured, dt):
        T_sp, P_sp = setpoints
        T, P, _ = measured
        outputs = []
        
        for i, (sp, meas, prev_err, integ) in enumerate([
            (T_sp, T, self.prev_errors[0], self.integrals[0]),
            (P_sp, P, self.prev_errors[1], self.integrals[1])
        ]):
            error = sp - meas
            integ += error * dt
            derivative = (error - prev_err) / dt
            output = self.Kp * error + self.Ki * integ + self.Kd * derivative
            self.prev_errors[i] = error
            self.integrals[i] = integ
            outputs.append(np.clip(output, 0, 2000 if i == 0 else 0.1))
        
        return outputs  # [Q, F]

# Fuzzy Logic Controller
class FuzzyController:
    def __init__(self):
        # Define fuzzy variables
        temp_error = ctrl.Antecedent(np.arange(-50, 51, 1), 'temp_error')
        press_error = ctrl.Antecedent(np.arange(-20, 21, 1), 'press_error')
        temp_change = ctrl.Antecedent(np.arange(-10, 11, 0.5), 'temp_change')
        heat_input = ctrl.Consequent(np.arange(0, 2001, 10), 'heat_input')
        flow_rate = ctrl.Consequent(np.arange(0, 0.11, 0.001), 'flow_rate')
        
        # Membership functions
        temp_error['negative'] = fuzz.trimf(temp_error.universe, [-50, -25, 0])
        temp_error['zero'] = fuzz.trimf(temp_error.universe, [-10, 0, 10])
        temp_error['positive'] = fuzz.trimf(temp_error.universe, [0, 25, 50])
        
        press_error['negative'] = fuzz.trimf(press_error.universe, [-20, -10, 0])
        press_error['zero'] = fuzz.trimf(press_error.universe, [-5, 0, 5])
        press_error['positive'] = fuzz.trimf(press_error.universe, [0, 10, 20])
        
        temp_change['negative'] = fuzz.trimf(temp_change.universe, [-10, -5, 0])
        temp_change['zero'] = fuzz.trimf(temp_change.universe, [-2, 0, 2])
        temp_change['positive'] = fuzz.trimf(temp_change.universe, [0, 5, 10])
        
        heat_input['low'] = fuzz.trimf(heat_input.universe, [0, 0, 1000])
        heat_input['medium'] = fuzz.trimf(heat_input.universe, [500, 1000, 1500])
        heat_input['high'] = fuzz.trimf(heat_input.universe, [1000, 2000, 2000])
        
        flow_rate['low'] = fuzz.trimf(flow_rate.universe, [0, 0, 0.05])
        flow_rate['medium'] = fuzz.trimf(flow_rate.universe, [0.03, 0.05, 0.07])
        flow_rate['high'] = fuzz.trimf(flow_rate.universe, [0.05, 0.1, 0.1])
        
        # Fuzzy rules
        rule1 = ctrl.Rule(temp_error['negative'] & temp_change['negative'], heat_input['high'])
        rule2 = ctrl.Rule(temp_error['zero'] & temp_change['zero'], heat_input['medium'])
        rule3 = ctrl.Rule(temp_error['positive'] & temp_change['positive'], heat_input['low'])
        rule4 = ctrl.Rule(press_error['negative'], flow_rate['high'])
        rule5 = ctrl.Rule(press_error['zero'], flow_rate['medium'])
        rule6 = ctrl.Rule(press_error['positive'], flow_rate['low'])
        
        # Control system
        self.system = ctrl.ControlSystem([rule1, rule2, rule3, rule4, rule5, rule6])
        self.sim = ctrl.ControlSystemSimulation(self.system)
        
    def control(self, setpoints, measured, prev_measured, dt):
        T_sp, P_sp = setpoints
        T, P, _ = measured
        T_prev, _, _ = prev_measured
        
        # Compute inputs
        self.sim.input['temp_error'] = T_sp - T
        self.sim.input['press_error'] = P_sp - P
        self.sim.input['temp_change'] = (T - T_prev) / dt
        
        # Compute outputs
        self.sim.compute()
        Q = self.sim.output['heat_input']
        F = self.sim.output['flow_rate']
        
        return [np.clip(Q, 0, 2000), np.clip(F, 0, 0.1)]

# Simulation and comparison
def run_simulation():
    reactor = ReactorModel()
    pid = PIDController()
    fuzzy = FuzzyController()
    
    # Time span
    t_span = np.linspace(0, 100, 1000)
    
    # Varying setpoints with disturbances
    def setpoint(t):
        T_sp = 350 + 10 * np.sin(0.1 * t) + 5 * np.random.normal(0, 1)
        P_sp = 100 + 5 * np.sin(0.05 * t) + 2 * np.random.normal(0, 1)
        return T_sp, P_sp
    
    # Simulate both controllers
    state_pid = np.zeros((3, len(t_span)))
    state_fuzzy = np.zeros((3, len(t_span)))
    inputs_pid = np.zeros((2, len(t_span)))
    inputs_fuzzy = np.zeros((2, len(t_span)))
    
    state_pid[:, 0] = [reactor.T0, reactor.P0, reactor.R0]
    state_fuzzy[:, 0] = [reactor.T0, reactor.P0, reactor.R0]
    
    for i in range(1, len(t_span)):
        dt = t_span[i] - t_span[i-1]
        T_sp, P_sp = setpoint(t_span[i-1])
        
        # PID control
        inputs_pid[:, i-1] = pid.control([T_sp, P_sp], state_pid[:, i-1], dt)
        state_pid[:, i] = reactor.simulate(
            [t_span[i-1], t_span[i]], 
            lambda t: inputs_pid[:, i-1], 
            state_pid[:, i-1]
        )[:, -1]
        
        # Fuzzy control
        inputs_fuzzy[:, i-1] = fuzzy.control(
            [T_sp, P_sp], 
            state_fuzzy[:, i-1], 
            state_fuzzy[:, max(0, i-2)], 
            dt
        )
        state_fuzzy[:, i] = reactor.simulate(
            [t_span[i-1], t_span[i]], 
            lambda t: inputs_fuzzy[:, i-1], 
            state_fuzzy[:, i-1]
        )[:, -1]
    
    # Calculate performance metrics
    T_sp, P_sp = zip(*[setpoint(t) for t in t_span])
    error_pid_T = np.abs(T_sp - state_pid[0])
    error_pid_P = np.abs(P_sp - state_pid[1])
    error_fuzzy_T = np.abs(T_sp - state_fuzzy[0])
    error_fuzzy_P = np.abs(P_sp - state_fuzzy[1])
    
    # Plot results
    plt.figure(figsize=(12, 10))
    
    plt.subplot(3, 1, 1)
    plt.plot(t_span, T_sp, 'k--', label='Setpoint')
    plt.plot(t_span, state_pid[0], 'b-', label='PID')
    plt.plot(t_span, state_fuzzy[0], 'r-', label='Fuzzy')
    plt.xlabel('Time (s)')
    plt.ylabel('Temperature (K)')
    plt.title('Temperature Control Comparison')
    plt.legend()
    plt.grid(True)
    
    plt
