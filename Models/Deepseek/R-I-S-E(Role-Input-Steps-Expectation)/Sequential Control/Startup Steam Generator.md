import casadi as ca
from model import steam_generator_model

class NMPCController:
    def __init__(self):
        self.N = 20          # Prediction horizon
        self.dt = 0.1        # Time step
        
        # States and inputs
        self.T = ca.MX.sym('T')
        self.P = ca.MX.sym('P')
        self.Q = ca.MX.sym('Q')
        
        # Parameters
        self.alpha = ca.MX.sym('alpha')
        self.beta = ca.MX.sym('beta')
        
        # Process model
        self.model = steam_generator_model()
        
        # Initial state
        self.Xk = ca.DM([300, 3])  # Initial temperature and pressure
        
        # Setpoint
        self.T_set = 520
        self.P_set = 30
        
        # Cost function
        cost = 0
        g = []
        Xk = self.Xk
        Uk = ca.MX.sym('Uk', 1)
        U = []
        for k in range(self.N):
            cost += (Xk[0] - self.T_set)**2 + (Xk[1] - self.P_set)**2
            
            # New NLP variable for the control
            Uk_end = Uk if k > 0 else ca.MX.sym('U_' + str(k), 1)
            
            # State at next time step
            X_next = self.model(Xk, Uk_end, ca.vertcat(self.alpha, self.beta))
            Xk = Xk + self.dt * X_next
            
            # Add inequality constraints on controls
            g.append(Uk_end >= 0)
            g.append(Uk_end <= 1000)
            
            # Add equality constraint for state update
            g.append(Xk[0] - 520 <= 0)  # Max temperature constraint
            g.append(Xk[1] - 30 >= 0)   # Min pressure constraint
            
            # Append new decision variables
            U.append(Uk_end)
        
        # Create an NLP solver
        nlp_prob = {'f': cost, 'g': ca.vertcat(*g), 'x': ca.vertcat(*U)}
        opts = {"ipopt.print_level": 0}
        self.solver = ca.nlpsol('solver', 'ipopt', nlp_prob, opts)
    
    def solve(self, current_state):
        self.Xk = current_state
        sol = self.solver(x0=ca.repmat(500, self.N, 1).flatten(), lbx=-ca.inf, ubx=ca.inf, lbg=0, ubg=0)
        u_opt = sol['x'].full().reshape((self.N, 1))
        return u_opt[0][0]
