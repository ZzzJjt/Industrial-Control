import numpy as np
import pandas as pd
from casadi import *
import matplotlib.pyplot as plt
from scipy.interpolate import interp1d

class CelluloseProductionMPC:
    def __init__(self):
        # Process parameters
        self.tank_capacity = 100  # tons
        self.initial_level = 60   # tons
        self.max_infeed = 50      # tons/hour (continuous feed max)
        self.batch_delay = 2      # hours
        self.batch_size = 30      # tons
        self.dt = 0.25           # hours (15 min control intervals)
        
        # MPC parameters
        self.horizon = 12         # 3 hour prediction horizon (12 steps)
        self.control_horizon = 8  # 2 hour control horizon
        
        # Initialize simulation
        self.tank_level = self.initial_level
        self.batch_queue = []
        self.time_hist = [0]
        self.level_hist = [self.initial_level]
        self.infeed_hist = [0]
        self.demand_hist = [0]
        
    def generate_demand_profile(self, total_time):
        """Create realistic stochastic demand pattern"""
        base_demand = 35  # tons/hour average
        time_points = np.linspace(0, total_time, 10)
        demand_points = base_demand + 15 * np.sin(2*np.pi*time_points/8) + \
                       5 * np.random.randn(len(time_points))
        demand_points = np.clip(demand_points, 20, 50)
        return interp1d(time_points, demand_points, kind='quadratic', fill_value="extrapolate")
    
    def process_batches(self, current_infeed):
        """Handle batch processing with delay"""
        # Add new material to queue
        batch_input = current_infeed * self.dt
        if batch_input > 0:
            self.batch_queue.append({'amount': batch_input, 'time_left': self.batch_delay})
        
        # Process batches
        total_released = 0
        for batch in self.batch_queue[:]:
            batch['time_left'] -= self.dt
            if batch['time_left'] <= 0:
                total_released += batch['amount']
                self.batch_queue.remove(batch)
        
        return total_released
    
    def tank_dynamics(self, infeed, demand):
        """Update tank level with batch processing"""
        released = self.process_batches(infeed)
        net_flow = released - demand * self.dt
        self.tank_level += net_flow
        self.tank_level = np.clip(self.tank_level, 0, self.tank_capacity)
        return self.tank_level
    
    def mpc_optimization(self, current_level, demand_forecast):
        """Solve MPC problem for optimal infeed rates"""
        opti = casadi.Opti()
        
        # Decision variables
        u = opti.variable(self.control_horizon)  # Infeed rates
        
        # Parameters
        level = opti.parameter()          # Current tank level
        demand = opti.parameter(self.horizon)  # Demand forecast
        
        # Simulation variables
        x = opti.variable(self.horizon + 1)  # Tank levels
        opti.subject_to(x[0] == level)       # Initial condition
        
        # Batch release model (simplified)
        batch_delay_steps = int(self.batch_delay / self.dt)
        released = vertcat(
            sum1(u[max(0, k-batch_delay_steps):k+1]) * (self.dt/batch_delay_steps)
            if k >= batch_delay_steps else 0
            for k in range(self.horizon)
        )
        
        # Tank dynamics constraints
        for k in range(self.horizon):
            # Use control horizon or last applied control
            uk = u[k] if k < self.control_horizon else u[-1]
            
            # Simplified batch release (would need refinement)
            release_k = released[k]
            
            # Tank balance
            opti.subject_to(x[k+1] == x[k] + release_k - demand[k]*self.dt)
        
        # Constraints
        opti.subject_to(0 <= u <= self.max_infeed)
        opti.subject_to(20 <= x <= 80)  # Operating range
        
        # Objective function
        J = 0
        for k in range(self.horizon):
            # Target level tracking (prefer 70% full)
            J += 10*(x[k+1] - 0.7*self.tank_capacity)**2
            
            # Minimize infeed changes (smooth operation)
            if k > 0:
                prev_u = u[k-1] if k-1 < self.control_horizon else u[-1]
                J += 5*(u[min(k, self.control_horizon-1)] - prev_u)**2
            
            # Minimize infeed rate (energy savings)
            J += 0.1*u[min(k, self.control_horizon-1)]**2
        
        # Solve
        opti.minimize(J)
        opti.set_value(level, current_level)
        opti.set_value(demand, demand_forecast[:self.horizon])
        
        opti.solver('ipopt')
        try:
            sol = opti.solve()
            return sol.value(u)[0]  # Return first control action
        except:
            print("MPC failed - using safe default")
            return min(self.max_infeed, 0.8*(self.tank_capacity - current_level)/self.dt)
    
    def simulate(self, total_time=24):
        """Run closed-loop simulation"""
        demand_func = self.generate_demand_profile(total_time)
        
        for t in np.arange(0, total_time, self.dt):
            # Current demand
            current_demand = demand_func(t)
            
            # Demand forecast (perfect for simplicity, could add noise)
            future_time = np.arange(t, t + self.horizon*self.dt, self.dt)
            demand_forecast = demand_func(future_time)
            
            # MPC control
            infeed = self.mpc_optimization(self.tank_level, demand_forecast)
            
            # Process dynamics
            new_level = self.tank_dynamics(infeed, current_demand)
            
            # Record history
            self.time_hist.append(t + self.dt)
            self.level_hist.append(new_level)
            self.infeed_hist.append(infeed)
            self.demand_hist.append(current_demand)
            
            # Print status
            print(f"Time {t:.2f}h: Level={new_level:.1f}t, Infeed={infeed:.1f}t/h, Demand={current_demand:.1f}t/h")
    
    def plot_results(self):
        """Visualize simulation results"""
        plt.figure(figsize=(12, 8))
        
        # Tank level
        plt.subplot(3, 1, 1)
        plt.plot(self.time_hist, self.level_hist, label='Tank Level')
        plt.axhline(80, color='r', linestyle='--', label='Upper Limit')
        plt.axhline(20, color='r', linestyle='--', label='Lower Limit')
        plt.ylabel('Tank Level (tons)')
        plt.legend()
        plt.grid(True)
        
        # Flows
        plt.subplot(3, 1, 2)
        plt.step(self.time_hist, self.infeed_hist, where='post', label='Infeed Rate')
        plt.plot(self.time_hist, self.demand_hist, label='Demand')
        plt.ylabel('Flow Rate (tons/h)')
        plt.legend()
        plt.grid(True)
        
        # Batch queue
        plt.subplot(3, 1, 3)
        batch_times = [t['time_left'] for t in self.batch_queue]
        batch_amounts = [t['amount'] for t in self.batch_queue]
        plt.barh(range(len(batch_times)), batch_amounts, tick_label=[f"{t:.1f}h" for t in batch_times])
        plt.xlabel('Batch Amount (tons)')
        plt.ylabel('Remaining Processing Time')
        plt.grid(True)
        
        plt.tight_layout()
        plt.show()

# Run simulation
process = CelluloseProductionMPC()
process.simulate(total_time=24)
process.plot_results()
