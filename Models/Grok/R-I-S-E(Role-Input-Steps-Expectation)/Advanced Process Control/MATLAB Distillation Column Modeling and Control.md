% Distillation Column MPC for Temperature and Composition Control
% No plots, numerical outputs only

clear all;
close all;

% 1. Mathematical Model (State-Space)
% States: x = [T; y] (T: tray temperature °C, y: top composition mole fraction)
% Input: u = F (feed rate kg/h)
% Disturbance: d = y_f (feed composition mole fraction)
% Model: dx/dt = A*x + B*u + E*d, y = C*x

% System parameters
A = [-0.1  0.05;  % Temperature decay and composition coupling
      0.02 -0.08]; % Composition dynamics
B = [0.5;  % Feed rate effect on temperature
     0.1]; % Feed rate effect on composition
E = [0.0;  % Disturbance effect on temperature
     0.2]; % Disturbance effect on composition
C = [1 0;  % Output: temperature
     0 1]; % Output: composition
D = zeros(2,1);

% Discretize model (Ts = 1 min)
Ts = 1;
sys_c = ss(A, [B E], C, D);
sys_d = c2d(sys_c, Ts);
Ad = sys_d.A;
Bd = sys_d.B(:,1);
Ed = sys_d.B(:,2);
Cd = sys_d.C;

% 2. MPC Parameters
Np = 10; % Prediction horizon
Nc = 3;  % Control horizon
nx = 2;  % Number of states
nu = 1;  % Number of inputs
ny = 2;  % Number of outputs

% Weighting matrices
Q = diag([10, 5]); % State weights (temperature, composition)
R = 0.1;           % Input weight
S = 0.01;          % Input rate weight

% Constraints
T_min = 50; T_max = 100; % Temperature (°C)
y_min = 0.8; y_max = 0.95; % Composition (mole fraction)
u_min = 0; u_max = 100;   % Feed rate (kg/h)
du_min = -10; du_max = 10; % Feed rate change (kg/h)

% Setpoints
r = [75; 0.9]; % [Temperature setpoint; Composition setpoint]

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

% Cost function: J = (Phi*x + Gamma*du - R)'*Q*(Phi*x + Gamma*du - R) + du'*R*du
Q_bar = kron(eye(Np), Q);
R_bar = kron(eye(Nc), R);
H = Gamma'*Q_bar*Gamma + R_bar;
f = @(x, r) Gamma'*Q_bar*(Phi*x - kron(ones(Np,1), r));

% Constraints: A_con*du <= b_con
% State constraints: y_min <= Cd*(Ad*x + Bd*u) <= y_max
% Input constraints: u_min <= u <= u_max, du_min <= du <= du_max
A_con = [];
b_con = [];
for i = 1:Np
    A_con = [A_con; Cd*Ad^i; -Cd*Ad^i]; % Output constraints
    b_con = [b_con; [T_max; y_max]; -[T_min; y_min]];
end
A_con = [A_con; eye(Nc); -eye(Nc)]; % Input rate constraints
b_con = [b_con; du_max*ones(Nc,1); -du_min*ones(Nc,1)];
A_con = [A_con; kron(eye(Nc), ones(1,Nc)); -kron(eye(Nc), ones(1,Nc))]; % Cumulative input constraints
u_prev = 0;

% 4. Simulation
n_sim = 100; % Simulation steps
x = [60; 0.85]; % Initial state: [T=60°C; y=0.85]
u = 50; % Initial feed rate (kg/h)
d = 0.8 * ones(n_sim,1); % Feed composition disturbance
d(50:end) = 0.85; % Step disturbance at t=50

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
    
    % MPC optimization
    f_k = f(x, r);
    H = 2*H; % quadprog expects 1/2*du'*H*du
    A_con_u = A_con;
    b_con_u = b_con;
    for i = 1:Np
        b_con_u((i-1)*ny+1:i*ny) = b_con((i-1)*ny+1:i*ny) - Phi((i-1)*ny+1:i*ny,:)*x;
    end
    b_con_u(2*Np*ny+1:2*Np*ny+Nc) = b_con(2*Np*ny+1:2*Np*ny+Nc) - u;
    b_con_u(2*Np*ny+Nc+1:end) = -b_con(2*Np*ny+Nc+1:end) + u;
    
    % Solve QP
    options = optimoptions('quadprog', 'Display', 'off');
    [du, ~, exitflag] = quadprog(H, f_k, A_con_u, b_con_u, [], [], [], [], [], options);
    
    if exitflag == 1
        u = u + du(1); % Apply first control move
        u = min(max(u, u_min), u_max); % Enforce input constraints
    else
        constraint_violation(k) = 1;
        u = u_prev; % Revert to previous input
    end
    
    % Update state
    x = Ad * x + Bd * u + Ed * d(k);
    u_prev = u;
end

% 5. Numerical Outputs
disp('Simulation Results:');
disp('Time (min) | Temp (°C) | Comp (mol frac) | Feed Rate (kg/h) | Constraint Violation');
for k = 1:10:n_sim
    fprintf('%10.1f | %9.2f | %14.3f | %15.2f | %20d\n', ...
        (k-1)*Ts, y_traj(1,k), y_traj(2,k), u_traj(k), constraint_violation(k));
end

disp('Final State:');
fprintf('Temperature: %.2f°C (Setpoint: %.2f°C)\n', y_traj(1,end), r(1));
fprintf('Composition: %.3f (Setpoint: %.3f)\n', y_traj(2,end), r(2));

disp('Performance Metrics:');
iae = sum(abs(r - y_traj), 2) * Ts;
fprintf('IAE Temperature: %.2f\n', iae(1));
fprintf('IAE Composition: %.2f\n', iae(2));
overshoot = max(max(r - y_traj, 0), [], 2);
fprintf('Overshoot Temperature: %.2f°C\n', overshoot(1));
fprintf('Overshoot Composition: %.3f\n', overshoot(2));
