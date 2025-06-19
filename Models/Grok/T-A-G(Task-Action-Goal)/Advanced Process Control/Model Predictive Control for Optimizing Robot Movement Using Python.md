import numpy as np

class RobotModel:
    def __init__(self):
        self.dt = 0.1  # Time step (s)
        self.v_max = 1.0  # Max linear velocity (m/s)
        self.omega_max = np.pi / 2  # Max angular velocity (rad/s)
        self.a_max = 0.5  # Max linear acceleration (m/s^2)
        self.alpha_max = np.pi  # Max angular acceleration (rad/s^2)
        self.r_robot = 0.2  # Robot radius (m)

    def simulate(self, x, u):
        # State: [x, y, theta], Control: [v, omega]
        x_next = np.zeros(3)
        x_next[0] = x[0] + u[0] * np.cos(x[2]) * self.dt
        x_next[1] = x[1] + u[0] * np.sin(x[2]) * self.dt
        x_next[2] = x[2] + u[1] * self.dt
        return x_next
