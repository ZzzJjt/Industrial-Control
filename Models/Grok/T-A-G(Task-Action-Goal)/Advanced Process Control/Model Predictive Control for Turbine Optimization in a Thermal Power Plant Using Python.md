import numpy as np
import matplotlib.pyplot as plt
from robot_model import RobotModel
from mpc_controller import MPCController

def generate_reference(t):
    # S-shaped reference trajectory
    x_ref = 10 * t / 100  # Linear x progression
    y_ref = 5 * np.sin(np.pi * t / 50)  # Sinusoidal y
    theta_ref = np.arctan2(np.diff(y_ref, prepend=y_ref[0]), np.diff(x_ref, prepend=x_ref[0]))
    return np.vstack((x_ref, y_ref, theta_ref))

def update_obstacles(t, static_obs, dynamic_obs):
    # Update dynamic obstacle positions
    obstacles = static_obs.copy()
    for i, (x0, y0, vx, vy, r) in enumerate(dynamic_obs):
        x = x0 + vx * t
        y = y0 + vy * t
        obstacles.append([x, y, r])
    return obstacles

def simulate():
    # Initialize model and controller
    model = RobotModel()
    controller = MPCController()
    
    # Simulation parameters
    T_sim = 100  # Simulation time (s)
    dt = model.dt
    N_sim = int(T_sim / dt) + 1
    t = np.linspace(0, T_sim, N_sim)
    
    # Initial state: [x, y, theta]
    x = np.array([0.0, 0.0, 0.0])
    
    # Reference trajectory
    x_ref = generate_reference(t)
    
    # Obstacles: [x, y, radius]
    static_obs = [
        [3.0, 2.0, 0.5],  # Static obstacle 1
        [7.0, -2.0, 0.5], # Static obstacle 2
        [5.0, 0.0, 0.5]   # Static obstacle 3
    ]
    dynamic_obs = [
        [2.0, 4.0, 0.1, -0.05, 0.5],  # Dynamic: start at (2,4), move left-down
        [8.0, -4.0, -0.1, 0.05, 0.5]  # Dynamic: start at (8,-4), move left-up
    ]
    
    # Storage
    x_traj = np.zeros((N_sim, 3))
    u_traj = np.zeros((N_sim-1, 2))
    x_traj[0, :] = x
    u_prev = np.array([0.0, 0.0])
    
    # Simulate
    for i in range(N_sim-1):
        # Update obstacles
        obstacles = update_obstacles(t[i], static_obs, dynamic_obs)
        
        # Compute control
        u = controller.control(x, x_ref[:, i:i+controller.Np], obstacles, u_prev)
        u_traj[i, :] = u
        u_prev = u
        
        # Simulate
        x = model.simulate(x, u)
        x_traj[i+1, :] = x
    
    # Performance metrics
    tracking_error = np.mean(np.sqrt((x_traj[:, 0] - x_ref[0, :])**2 + (x_traj[:, 1] - x_ref[1, :])**2))
    energy = np.sum(np.abs(u_traj[:, 0]) + np.abs(u_traj[:, 1])) * dt
    collisions = 0
    for i in range(N_sim):
        for obs in static_obs + update_obstacles(t[i], [], dynamic_obs):
            dist = np.sqrt((x_traj[i, 0] - obs[0])**2 + (x_traj[i, 1] - obs[1])**2)
            if dist < model.r_robot + obs[2]:
                collisions += 1
    
    print(f"Tracking Error: {tracking_error:.3f} m")
    print(f"Energy Usage: {energy:.3f} (v+ω)·s")
    print(f"Collisions: {collisions}")

    # Plot results
    plt.figure(figsize=(12, 8))
    
    # Trajectory plot
    plt.subplot(2, 1, 1)
    plt.plot(x_traj[:, 0], x_traj[:, 1], 'b-', label='Robot Path')
    plt.plot(x_ref[0, :], x_ref[1, :], 'r--', label='Reference Path')
    for obs in static_obs:
        circle = plt.Circle((obs[0], obs[1]), obs[2], color='k', alpha=0.3)
        plt.gca().add_patch(circle)
    for i in range(0, N_sim, 100):  # Plot dynamic obstacles at intervals
        for obs in update_obstacles(t[i], [], dynamic_obs):
            circle = plt.Circle((obs[0], obs[1]), obs[2], color='gray', alpha=0.1)
            plt.gca().add_patch(circle)
    plt.xlabel('X (m)')
    plt.ylabel('Y (m)')
    plt.title('Robot Trajectory')
    plt.legend()
    plt.grid(True)
    plt.axis('equal')
    
    # Control inputs
    plt.subplot(2, 1, 2)
    plt.plot(t[:-1], u_traj[:, 0], label='Linear Velocity (m/s)')
    plt.plot(t[:-1], u_traj[:, 1], label='Angular Velocity (rad/s)')
    plt.xlabel('Time (s)')
    plt.ylabel('Control Inputs')
    plt.title('Control Inputs')
    plt.legend()
    plt.grid(True)
    
    plt.tight_layout()
    plt.savefig('robot_mpc_trajectory.png')
    plt.show()

if __name__ == "__main__":
    simulate()
