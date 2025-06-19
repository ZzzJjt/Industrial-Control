% Aircraft Trajectory Optimization using MPC
% 2D Longitudinal Model, Numerical Outputs Only

clear all;
close all;

% 1. Aircraft Dynamics Model (State-Space)
% States: x = [x; h; Vx; Vz; theta] (position m, altitude m, velocities m/s, pitch rad)
% Inputs: u = [T; delta_e] (thrust N, elevator angle rad)
% Disturbance: d = [Wx; Wz] (wind velocities m/s)
% Model: dx/dt = A*x + B*u + E*d, y = C*x

% Aircraft parameters
m = 5000; % Mass (kg)
S = 20;   % Wing area (m^2)
rho = 1.225; % Air density (kg/m^3)
g = 9.81; % Gravity (m/s^2)
CL0 = 0.5; % Lift coefficient
CD0 = 0.02; % Drag coefficient
Cm = -0.1; % Pitch moment coefficient
k = 0.05; % Control effectiveness

% Continuous-time state-space matrices
A = [0 0 1 0 0;
     0 0 0 1 0;
     0 0 -CD0*rho*S/(2*m) 0 0;
     0 0 0 -CD0*rho*S/(2*m) -g;
     0 0 0 CL0*rho*S/(2*m) 0];
B = [0 0;
     0 0;
     1/m 0;
     0 k/m;
     0 Cm];
E = [0 0;
     0 0;
     -1/m 0;
     0 -1/m;
     0 0];
C = [1 0 0 0 0; % Output: x
     0 1 0 0 0; % Output: h
     0 0 0 0 1]; % Output: theta
D = zeros(3,2);

% Discretize model (Ts = 1 s)
Ts = 1;
sys_c = ss(A, [B E], C, D);
sys_d = c2d(sys_c, Ts);
Ad = sys_d.A;
Bd = sys_d.B(:,1:2);
Ed = sys_d.B(:,3:4);
Cd = sys_d.C;

% 2. MPC Formulation
Np = 10; % Prediction horizon
Nc = 3;  % Control horizon
nx = 5;  % Number of states
nu = 2;  % Number of inputs
ny = 3;  % Number of outputs

% Weighting matrices
Q = diag([1, 10, 0.1]); % Weights for x, h, theta
R = diag([0.5, 0.1]);   % Weights for T, delta_e (penalize thrust for fuel)
S = diag([0.01, 0.01]); % Weights for input rates

% Constraints
h_min = 1000; h_max = 10000; % Altitude (m)
theta_min = -0.2; theta_max = 0.2; % Pitch (rad)
T_min = 0; T_max = 50000; % Thrust (N)
de_min = -0.3; de_max = 0.3; % Elevator angle (rad)
dT_min = -5000; dT_max = 5000; % Thrust rate (N/s)
dde_min = -0.05; dde_max = 0.05; % Elevator rate (rad/s)

% Reference trajectory (setpoints for x, h, theta)
r = @(t) [1000*t; 5000; 0]; % Linear x, constant altitude 5000m, level flight

% 3. MPC Matrices
% Prediction matrices
Phi = zeros(Np*ny, nx);
Gamma = zeros(Np*ny, Nc*nu);
for i = 1:Np
    Phi((i-1)*ny+1:i*ny, :) = Cd * Ad^i;
    for j = 1:min(i, Nc)
        Gamma((i-1)*ny+1:i*ny, (j-1)*nu+1:j*nu) = Cd * Ad^(i-j) * Bd;
    end
end

% Cost function: J = (Phi*x + Gamma*du - R)'*Q*(Phi*x + Gamma*du - R) + du'*S*du
Q_bar = kron(eye(Np), Q);
S_bar = kron(eye(Nc), S);
H = Gamma'*Q_bar*Gamma + S_bar;
f = @(x, r_t) Gamma'*Q_bar*(Phi*x - kron(ones(Np,1), r_t));

% Constraints: A_con*du <= b_con
A_con = [];
b_con = [];
for i = 1:Np
    A_con = [A_con; Cd*Ad^i; -Cd*Ad^i]; % Output constraints
    b_con = [b_con; [inf; h_max; theta_max]; -[inf; h_min; theta_min]];
end
A_con = [A_con; eye(Nc*nu); -eye(Nc*nu)]; % Input rate constraints
b_con = [b_con; [dT_max; dde_max]*ones(Nc,1); -[dT_min; dde_min]*ones(Nc,1)];
A_con = [A_con; kron(eye(Nc), eye(nu)); -kron(eye(Nc), eye(nu))]; % Cumulative input constraints

% 4. Simulation
n_sim = 500; % Simulation steps (500s)
x = [0; 4000; 200; 0; 0]; % Initial state: [x=0m, h=4000m, Vx=200m/s, Vz=0, theta=0]
u = [20000; 0]; % Initial inputs: [T=20000N, delta_e=0]
d = zeros(n_sim, 2); % Wind disturbance
d(100:200,1) = 10; % Wind gust +10m/s in x at t=100–200s
d(300:400,2) = -5; % Wind gust -5m/s in z at t=300–400s

% Storage
x_traj = zeros(nx, n_sim);
u_traj = zeros(nu, n_sim);
y_traj = zeros(ny, n_sim);
constraint_violation = zeros(n_sim, 1);

for k = 1:n_sim
    % Current state and output
    y = Cd * x;
    x_traj(:,k) = x;
    y_traj(:,k) = y;
    u_traj(:,k) = u;
    
    % Reference at current time
    r_t = r((k-1)*Ts);
    
    % MPC optimization
    f_k = f(x, r_t);
    H_qp = 2*H; % quadprog expects 1/2*du'*H*du
    A_con_u = A_con;
    b_con_u = b_con;
    for i = 1:Np
        b_con_u((i-1)*ny+1:i*ny) = b_con((i-1)*ny+1:i*ny) - Phi((i-1)*ny+1:i*ny,:)*x;
    end
    b_con_u(2*Np*ny+1:2*Np*ny+Nc*nu) = b_con(2*Np*ny+1:2*Np*ny+Nc*nu) - kron(ones(Nc,1), u);
    b_con_u(2*Np*ny+Nc*nu+1:end) = -b_con(2*Np*ny+Nc*nu+1:end) + kron(ones(Nc,1), u);
    
    % Solve QP
    options = optimoptions('quadprog', 'Display', 'off');
    [du, ~, exitflag] = quadprog(H_qp, f_k, A_con_u, b_con_u, [], [], [], [], [], options);
    
    if exitflag == 1
        u = u + du(1:nu); % Apply first control move
        u = min(max(u, [T_min; de_min]), [T_max; de_max]); % Enforce input constraints
    else
        constraint_violation(k) = 1;
        u = u; % Retain previous input
    end
    
    % Update state
    x = Ad * x + Bd * u + Ed * d(k,:)';
end

% 5. Numerical Outputs
disp('Simulation Results:');
disp('Time (s) | X (m) | H (m) | Theta (rad) | Thrust (N) | Elevator (rad) | Constraint Violation');
for k = 1:50:n_sim
    fprintf('%8.1f | %8.1f | %8.1f | %11.3f | %10.1f | %12.3f | %20d\n', ...
        (k-1)*Ts, y_traj(1,k), y_traj(2,k), y_traj(3,k), u_traj(1,k), u_traj(2,k), constraint_violation(k));
end

disp('Final State:');
fprintf('X: %.1fm (Setpoint: %.1fm)\n', y_traj(1,end), r((n_sim-1)*Ts)(1));
fprintf('H: %.1fm (Setpoint: %.1fm)\n', y_traj(2,end), r((n_sim-1)*Ts)(2));
fprintf('Theta: %.3frad (Setpoint: %.3frad)\n', y_traj(3,end), r((n_sim-1)*Ts)(3));

disp('Performance Metrics:');
iae = sum(abs(r((0:n_sim-1)*Ts)' - y_traj), 2) * Ts;
fprintf('IAE X: %.1f m\n', iae(1));
fprintf('IAE H: %.1f m\n', iae(2));
fprintf('IAE Theta: %.3f rad\n', iae(3));
fuel = sum(u_traj(1,:)) * Ts / 3600; % Approximate fuel (thrust*time)
fprintf('Fuel Consumption: %.1f kN·h\n', fuel);
fprintf('Constraint Violations: %d\n', sum(constraint_violation));
