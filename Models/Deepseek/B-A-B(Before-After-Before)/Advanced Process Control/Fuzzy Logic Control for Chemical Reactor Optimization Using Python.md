import numpy as np
import matplotlib.pyplot as plt
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import pandas as pd

# 1. Define Antecedents (Inputs) and Consequents (Outputs)
def setup_fuzzy_system():
    # Input variables
    temp_error = ctrl.Antecedent(np.arange(-50, 51, 1), 'temperature_error')
    pressure = ctrl.Antecedent(np.arange(0, 2.1, 0.1), 'pressure')
    reaction_rate = ctrl.Antecedent(np.arange(0, 101, 1), 'reaction_rate')
    
    # Output variable
    coolant_flow = ctrl.Consequent(np.arange(0, 101, 1), 'coolant_flow')
    valve_opening = ctrl.Consequent(np.arange(0, 101, 1), 'valve_opening')
    
    # 2. Define fuzzy membership functions
    # Temperature error (difference from setpoint)
    temp_error['negative'] = fuzz.trapmf(temp_error.universe, [-50, -50, -30, -10])
    temp_error['optimal'] = fuzz.trimf(temp_error.universe, [-20, 0, 20])
    temp_error['positive'] = fuzz.trapmf(temp_error.universe, [10, 30, 50, 50])
    
    # Pressure
    pressure['low'] = fuzz.trapmf(pressure.universe, [0, 0, 0.7, 1.0])
    pressure['normal'] = fuzz.trimf(pressure.universe, [0.8, 1.2, 1.6])
    pressure['high'] = fuzz.trapmf(pressure.universe, [1.4, 1.7, 2.0, 2.0])
    
    # Reaction rate
    reaction_rate['slow'] = fuzz.trapmf(reaction_rate.universe, [0, 0, 30, 50])
    reaction_rate['medium'] = fuzz.trimf(reaction_rate.universe, [40, 60, 80])
    reaction_rate['fast'] = fuzz.trapmf(reaction_rate.universe, [70, 90, 100, 100])
    
    # Coolant flow output
    coolant_flow['low'] = fuzz.trapmf(coolant_flow.universe, [0, 0, 30, 50])
    coolant_flow['medium'] = fuzz.trimf(coolant_flow.universe, [40, 60, 80])
    coolant_flow['high'] = fuzz.trapmf(coolant_flow.universe, [70, 90, 100, 100])
    
    # Valve opening output
    valve_opening['closed'] = fuzz.trapmf(valve_opening.universe, [0, 0, 30, 50])
    valve_opening['half'] = fuzz.trimf(valve_opening.universe, [40, 60, 80])
    valve_opening['open'] = fuzz.trapmf(valve_opening.universe, [70, 90, 100, 100])
    
    return temp_error, pressure, reaction_rate, coolant_flow, valve_opening

# 3. Create fuzzy rules based on process knowledge
def create_control_rules(temp_error, pressure, reaction_rate, coolant_flow, valve_opening):
    rules = [
        # Temperature control rules
        ctrl.Rule(temp_error['negative'] & pressure['normal'], (coolant_flow['low'], valve_opening['half'])),
        ctrl.Rule(temp_error['optimal'], (coolant_flow['medium'], valve_opening['half'])),
        ctrl.Rule(temp_error['positive'] & pressure['normal'], (coolant_flow['high'], valve_opening['half'])),
        
        # Pressure compensation rules
        ctrl.Rule(pressure['high'], (coolant_flow['high'], valve_opening['open'])),
        ctrl.Rule(pressure['low'], (coolant_flow['low'], valve_opening['closed'])),
        
        # Reaction rate adjustment rules
        ctrl.Rule(reaction_rate['fast'] & temp_error['positive'], (coolant_flow['high'], valve_opening['half'])),
        ctrl.Rule(reaction_rate['slow'] & temp_error['negative'], (coolant_flow['low'], valve_opening['closed'])),
        
        # Combined conditions
        ctrl.Rule(temp_error['positive'] & pressure['high'], (coolant_flow['high'], valve_opening['open'])),
        ctrl.Rule(temp_error['negative'] & pressure['low'], (coolant_flow['low'], valve_opening['closed']))
    ]
    
    control_system = ctrl.ControlSystem(rules)
    return ctrl.ControlSystemSimulation(control_system)

# 4. Reactor Simulation with Fuzzy Control
class ReactorSimulation:
    def __init__(self, fuzzy_ctrl):
        self.fuzzy = fuzzy_ctrl
        self.history = {
            'time': [],
            'temperature': [],
            'pressure': [],
            'reaction_rate': [],
            'coolant_flow': [],
            'valve_opening': [],
            'setpoint': []
        }
        self.current_state = {
            'temperature': 150,
            'pressure': 1.0,
            'reaction_rate': 50
        }
    
    def update_state(self, t, setpoint):
        # Apply fuzzy control
        self.fuzzy.input['temperature_error'] = setpoint - self.current_state['temperature']
        self.fuzzy.input['pressure'] = self.current_state['pressure']
        self.fuzzy.input['reaction_rate'] = self.current_state['reaction_rate']
        self.fuzzy.compute()
        
        # Get control actions
        coolant = self.fuzzy.output['coolant_flow']
        valve = self.fuzzy.output['valve_opening']
        
        # Simulate process dynamics (simplified)
        self.current_state['temperature'] += 0.1*(setpoint - self.current_state['temperature']) - 0.05*coolant
        self.current_state['pressure'] = max(0.8, min(1.8, self.current_state['pressure'] + 0.01*valve - 0.02*coolant))
        self.current_state['reaction_rate'] = max(10, min(90, 50 + 0.5*(self.current_state['temperature']-150) + np.random.normal(0, 3)))
        
        # Record history
        self.history['time'].append(t)
        self.history['temperature'].append(self.current_state['temperature'])
        self.history['pressure'].append(self.current_state['pressure'])
        self.history['reaction_rate'].append(self.current_state['reaction_rate'])
        self.history['coolant_flow'].append(coolant)
        self.history['valve_opening'].append(valve)
        self.history['setpoint'].append(setpoint)
        
        return self.current_state

# 5. Performance Visualization
def plot_results(simulation):
    fig, axs = plt.subplots(3, 1, figsize=(12, 10))
    
    # Temperature plot
    axs[0].plot(simulation.history['time'], simulation.history['temperature'], label='Actual')
    axs[0].plot(simulation.history['time'], simulation.history['setpoint'], 'r--', label='Setpoint')
    axs[0].set_ylabel('Temperature (°C)')
    axs[0].legend()
    axs[0].grid(True)
    
    # Pressure plot
    axs[1].plot(simulation.history['time'], simulation.history['pressure'])
    axs[1].set_ylabel('Pressure (bar)')
    axs[1].grid(True)
    
    # Control actions plot
    axs[2].plot(simulation.history['time'], simulation.history['coolant_flow'], label='Coolant Flow')
    axs[2].plot(simulation.history['time'], simulation.history['valve_opening'], label='Valve Opening')
    axs[2].set_xlabel('Time (min)')
    axs[2].set_ylabel('Control Output (%)')
    axs[2].legend()
    axs[2].grid(True)
    
    plt.tight_layout()
    plt.show()

# Main execution
if __name__ == "__main__":
    # Setup fuzzy system
    temp_err, pressure, rate, coolant, valve = setup_fuzzy_system()
    fuzzy_system = create_control_rules(temp_err, pressure, rate, coolant, valve)
    
    # Initialize simulation
    reactor = ReactorSimulation(fuzzy_system)
    
    # Run simulation with varying conditions
    for t in range(120):  # 2 hours simulation
        # Vary setpoint and introduce disturbances
        if t < 30:
            setpoint = 150
        elif t < 60:
            setpoint = 160 + 5*np.sin(t/5)
        elif t < 90:
            setpoint = 140
        else:
            setpoint = 155
            
        reactor.update_state(t, setpoint)
    
    # Visualize results
    plot_results(reactor)
    
    # Calculate control performance
    temp_errors = np.array(reactor.history['setpoint']) - np.array(reactor.history['temperature'])
    print(f"Mean Absolute Temperature Error: {np.mean(np.abs(temp_errors)):.2f}°C")
    print(f"Maximum Temperature Deviation: {np.max(np.abs(temp_errors)):.2f}°C")
    print(f"Pressure maintained between {min(reactor.history['pressure']):.2f}-{max(reactor.history['pressure']):.2f} bar")
