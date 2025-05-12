
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import minimize
from scipy.integrate import odeint

class TurbineMPC:
    def __init__(self):
        # Turbine system parameters
        self.thermal_time_constant = 30.0  # seconds
        self.rotational_inertia = 5000.0  # kg·m²
        self.max_power_output = 300.0  # MW
        self.min_power_output = 50.0  # MW
        self.max_ramp_rate = 10.0  # MW/s
        self.steam_temp_nominal = 540.0  # °C
        self.steam_pressure_nominal = 16.5  # MPa
        
        # MPC parameters
        self.dt = 1.0  # time step [s]
        self.horizon = 20  # prediction horizon steps
        self.control_horizon = 10  # control horizon steps
        
        # Cost function weights
        self.w_power = 1.0  # Power tracking weight
        self.w_steam = 0.5  # Steam condition stability weight
        self.w_ramp = 0.3  # Ramp rate penalty weight
        self.w_efficiency = 0.7  # Efficiency optimization weight
        
        # Current state [power_output, steam_temp, steam_pressure, rotor_speed]
        self.state = np.array([200.0, self.steam_temp_nominal, 
                              self.steam_pressure_nominal, 3000.0])
        
        # Operational constraints
        self.power_demand = 200.0  # Current demand [MW]
        self.disturbances = np.zeros(self.horizon)  # Future disturbances
        
    def turbine_dynamics(self, state, control, disturbance):
        """Nonlinear turbine dynamics model"""
        power, temp, pressure, speed = state
        valve_position, fuel_rate = control
        
        # Power output dynamics
        dPdt = (valve_position * self.max_power_output - power) / self.thermal_time_constant
        
        # Steam temperature dynamics
        dTdt = (fuel_rate * (self.steam_temp_nominal + 50) - temp) / (self.thermal_time_constant * 0.8)
        
        # Steam pressure dynamics
        dpdt = (valve_position * self.steam_pressure_nominal - pressure) / (self.thermal_time_constant * 0.7)
        
        # Rotor speed dynamics (simplified)
        dsdt = (power * 1000 - disturbance - self.state[0] * 1000) / self.rotational_inertia
        
        return np.array([dPdt, dTdt, dpdt, dsdt])
    
    def predict_trajectory(self, init_state, controls, disturbances):
        """Predict system evolution given control sequence"""
        trajectory = []
        state = init_state.copy()
        trajectory.append(state)
        
        for i in range(self.horizon):
            # Get current control (zero-order hold beyond control horizon)
            ctrl = controls[min(i, self.control_horizon-1)*2:(min(i, self.control_horizon-1)+1)*2]
            dist = disturbances[i]
            
            # Integrate dynamics for one time step
            state = state + self.dt * self.turbine_dynamics(state, ctrl, dist)
            
            # Apply constraints
            state[0] = np.clip(state[0], self.min_power_output, self.max_power_output)
            state[1] = np.clip(state[1], self.steam_temp_nominal-20, self.steam_temp_nominal+20)
            state[2] = np.clip(state[2], self.steam_pressure_nominal*0.9, self.steam_pressure_nominal*1.1)
            state[3] = np.clip(state[3], 2950, 3050)  # RPM constraints
            
            trajectory.append(state)
            
        return np.array(trajectory)
    
    def cost_function(self, controls, power_demand, disturbances):
        """MPC cost function calculation"""
        cost = 0.0
        trajectory = self.predict_trajectory(self.state, controls, disturbances)
        
        # Power tracking cost
        for i in range(1, self.horizon+1):
            power_error = trajectory[i, 0] - power_demand
            cost += self.w_power * power_error**2
            
            # Efficiency cost (higher temp/pressure generally more efficient)
            temp_dev = (trajectory[i, 1] - self.steam_temp_nominal) / 20.0
            press_dev = (trajectory[i, 2] - self.steam_pressure_nominal) / 1.5
            cost -= self.w_efficiency * (0.5*temp_dev + 0.5*press_dev)  # Negative because we want to maximize
            
            # Steam condition stability
            cost += self.w_steam * (temp_dev**2 + press_dev**2)
            
            # Ramp rate penalty
            if i > 1:
                ramp_rate = (trajectory[i, 0] - trajectory[i-1, 0]) / self.dt
                cost += self.w_ramp * min(0, (self.max_ramp_rate - abs(ramp_rate)))**2
        
        # Control effort penalty
        for i in range(self.control_horizon):
            cost += 0.01 * controls[i*2]**2  # Valve position
            cost += 0.02 * controls[i*2+1]**2  # Fuel rate
            
        return cost
    
    def update_demand_and_disturbances(self, step):
        """Simulate changing demand and disturbances"""
        # Simulate changing power demand
        if step < 50:
            self.power_demand = 200.0
        elif step < 100:
            self.power_demand = 180.0 + 20 * np.sin((step-50)/10)
        else:
            self.power_demand = 220.0
            
        # Simulate disturbances (e.g., grid frequency variations)
        self.disturbances = 5000 * np.sin(np.linspace(0, 2*np.pi, self.horizon) * (0.5 + 0.5*np.sin(step/20))
        
        return self.power_demand, self.disturbances
    
    def solve_mpc(self, power_demand, disturbances):
        """Solve MPC optimization problem"""
        # Initial guess (current operating point)
        init_controls = np.zeros(2 * self.control_horizon)
        init_controls[::2] = self.state[0] / self.max_power_output  # Valve position
        init_controls[1::2] = 0.8  # Fuel rate
        
        # Control bounds
        bounds = []
        for i in range(self.control_horizon):
            bounds.append((0.1, 1.0))  # Valve position (10-100%)
            bounds.append((0.5, 1.2))  # Fuel rate (50-120%)
        
        # Solve optimization
        result = minimize(self.cost_function, init_controls,
                        args=(power_demand, disturbances),
                        bounds=bounds, method='SLSQP')
        
        return result.x
    
    def control_step(self, step):
        """Execute one MPC control step"""
        # Update demand and disturbance forecast
        power_demand, disturbances = self.update_demand_and_disturbances(step)
        
        # Solve MPC problem
        controls = self.solve_mpc(power_demand, disturbances)
        
        # Apply first control
        control = controls[:2]
        self.state = self.state + self.dt * self.turbine_dynamics(self.state, control, disturbances[0])
        
        # Apply constraints to current state
        self.state[0] = np.clip(self.state[0], self.min_power_output, self.max_power_output)
        self.state[1] = np.clip(self.state[1], self.steam_temp_nominal-20, self.steam_temp_nominal+20)
        self.state[2] = np.clip(self.state[2], self.steam_pressure_nominal*0.9, self.steam_pressure_nominal*1.1)
        self.state[3] = np.clip(self.state[3], 2950, 3050)
        
        return self.state, controls, power_demand

def simulate_turbine_control():
    """Run turbine control simulation"""
    turbine = TurbineMPC()
    sim_steps = 150
    
    # Data logging
    power_output = []
    steam_temp = []
    steam_pressure = []
    rotor_speed = []
    power_demand = []
    valve_positions = []
    fuel_rates = []
    
    for step in range(sim_steps):
        state, controls, demand = turbine.control_step(step)
        
        # Store data
        power_output.append(state[0])
        steam_temp.append(state[1])
        steam_pressure.append(state[2])
        rotor_speed.append(state[3])
        power_demand.append(demand)
        valve_positions.append(controls[0])
        fuel_rates.append(controls[1])
    
    # Plot results
    plot_turbine_performance(sim_steps, power_output, steam_temp, 
                            steam_pressure, rotor_speed, power_demand,
                            valve_positions, fuel_rates)

def plot_turbine_performance(steps, power, temp, pressure, speed, demand, valve, fuel):
    """Visualize turbine performance under MPC control"""
    plt.figure(figsize=(12, 10))
    
    # Power output and demand
    plt.subplot(3, 1, 1)
    plt.plot(range(steps), power, 'b-', label='Actual Power')
    plt.plot(range(steps), demand, 'r--', label='Demand')
    plt.ylabel('Power Output (MW)')
    plt.title('Turbine Performance with MPC Control')
    plt.legend()
    plt.grid(True)
    
    # Steam conditions
    plt.subplot(3, 1, 2)
    plt.plot(range(steps), temp, 'g-', label='Steam Temp')
    plt.plot(range(steps), pressure, 'm-', label='Steam Pressure')
    plt.ylabel('Temperature (°C) / Pressure (MPa)')
    plt.legend()
    plt.grid(True)
    
    # Controls and speed
    plt.subplot(3, 1, 3)
    ax1 = plt.gca()
    ax1.plot(range(steps), valve, 'c-', label='Valve Position')
    ax1.plot(range(steps), fuel, 'y-', label='Fuel Rate')
    ax1.set_xlabel('Time (s)')
    ax1.set_ylabel('Control Inputs')
    ax1.legend(loc='upper left')
    ax1.grid(True)
    
    ax2 = ax1.twinx()
    ax2.plot(range(steps), speed, 'k--', label='Rotor Speed')
    ax2.set_ylabel('Speed (RPM)')
    ax2.legend(loc='upper right')
    
    plt.tight_layout()
    plt.show()

if __name__ == "__main__":
    simulate_turbine_control()
