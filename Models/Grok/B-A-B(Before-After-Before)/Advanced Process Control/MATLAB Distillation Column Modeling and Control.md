% Distillation Column Model and MPC Controller
% Parameters
N = 10; % Number of trays
V = 2.0; % Vapor flow rate (kmol/min)
L = 1.8; % Liquid flow rate (kmol/min)
F = 1.0; % Feed flow rate (kmol/min)
zF = 0.5; % Feed composition (mole fraction)
alpha = 2.5; % Relative volatility
xB_init = 0.1; % Bottom composition
xD_init = 0.9; % Distillate composition
T_ref = 350; % Reference temperature (K)
dt = 0.1; % Time step (min)
N_sim = 100; % Simulation steps

% State-space model matrices
A = zeros(N);
B = zeros(N, 2);
C = eye(N);
D = zeros(N, 2);

% Tray dynamics (simplified mass balance)
for i = 1:N
    if i == 1 % Bottom tray
        A(i, i) = -L - V;
        A(i, i+1) = V;
        B(i, 1) = 1; % Feed rate input
    elseif i == N % Top tray
        A(i, i-1) = L;
        A(i, i) = -L - V;
        B(i, 2) = 1; % Reflux rate input
    else % Intermediate trays
        A(i, i-1) = L;
        A(i, i) = -L - V;
        A(i, i+1) = V;
    end
end
A = A / (1 + alpha); % Incorporate volatility
B = B / (1 + alpha);

% Discretize the system
sys_c = ss(A, B, C, D);
sys_d = c2d(sys_c, dt, 'zoh');
Ad = sys_d.A;
Bd = sys_d.B;
Cd = sys_d.C;

% MPC Parameters
Np = 10; % Prediction horizon
Nc = 3; % Control horizon
Q = eye(N); % State weighting matrix
R = 0.1 * eye(2); % Input weighting matrix
x0 = linspace(xB_init, xD_init, N)'; % Initial composition profile
u0 = [F; L]; % Initial inputs
u_min = [0.5; 1.0]; % Minimum feed and reflux rates
u_max = [1.5; 2.5]; % Maximum feed and reflux rates
xD_sp = 0.95; % Distillate composition setpoint
xB_sp = 0.05; % Bottom composition setpoint

% Temperature model (simplified relation to composition)
T = @(x) 340 + 20 * (x(1) + x(N)); % Bottom and top tray composition to temperature

% Simulation storage
x = zeros(N, N_sim);
u = zeros(2, N_sim);
x(:, 1) = x0;
u(:, 1) = u0;
T_out = zeros(1, N_sim);
cost = zeros(1, N_sim);

% MPC optimization function
function [u_opt, J] = mpc_optimize(xk, Ad, Bd, Cd, Np, Nc, Q, R, u_prev, u_min, u_max, sp)
    n = size(Ad, 1);
    m = size(Bd, 2);
    H = zeros(Nc*m, Nc*m);
    f = zeros(Nc*m, 1);
    A_pred = zeros(Np*n, Nc*m);
    b_pred = zeros(Np*n, 1);
    
    % Build prediction matrices
    Phi = zeros(Np*n, n);
    Theta = zeros(Np*n, Nc*m);
    Phi(1:n, :) = Ad;
    for i = 2:Np
        Phi(i*n-(n-1):i*n, :) = Phi((i-1)*n-(n-1):(i-1)*n, :) * Ad;
    end
    
    for i = 1:Np
        for j = 1:min(i, Nc)
            if i-j >= 0
                Theta(i*n-(n-1):i*n, j*m-(m-1):j*m) = Phi((i-j)*n-(n-1):(i-j)*n, :) * Bd;
            end
        end
    end
    
    % Cost function matrices
    for i = 1:Np
        H = H + Theta(i*n-(n-1):i*n, :)' * Q * Theta(i*n-(n-1):i*n, :);
        f = f + Theta(i*n-(n-1):i*n, :)' * Q * (Phi(i*n-(n-1):i*n, :) * xk - sp);
    end
    H = H + kron(eye(Nc), R);
    
    % Constraints
    A_ineq = [kron(eye(Nc), eye(m)); -kron(eye(Nc), eye(m))];
    b_ineq = [repmat(u_max, Nc, 1); -repmat(u_min, Nc, 1)];
    
    % Quadratic programming
    H = (H + H') / 2; % Ensure symmetry
    options = optimoptions('quadprog', 'Display', 'off');
    du = quadprog(H, f, A_ineq, b_ineq, [], [], [], [], [], options);
    u_opt = u_prev + du(1:m);
    J = 0.5 * du' * H * du + f' * du;
end

% Main simulation loop
for k = 1:N_sim-1
    % Setpoint for top and bottom compositions
    sp = zeros(N, 1);
    sp(1) = xB_sp;
    sp(N) = xD_sp;
    
    % Compute MPC control action
    [u_opt, J] = mpc_optimize(x(:, k), Ad, Bd, Cd, Np, Nc, Q, R, u(:, k), u_min, u_max, sp);
    u(:, k+1) = u_opt;
    cost(k) = J;
    
    % Update state
    x(:, k+1) = Ad * x(:, k) + Bd * u(:, k+1);
    
    % Compute temperature
    T_out(k) = T(x(:, k));
end
T_out(N_sim) = T(x(:, N_sim));

% Output results
fprintf('Final Distillate Composition: %.3f (Setpoint: %.3f)\n', x(N, end), xD_sp);
fprintf('Final Bottom Composition: %.3f (Setpoint: %.3f)\n', x(1, end), xB_sp);
fprintf('Final Temperature: %.2f K (Reference: %.2f K)\n', T_out(end), T_ref);
fprintf('Average Control Cost: %.2f\n', mean(cost));
fprintf('Final Feed Rate: %.2f kmol/min\n', u(1, end));
fprintf('Final Reflux Rate: %.2f kmol/min\n', u(2, end));

% Check constraint satisfaction
if all(u(1, :) >= u_min(1)) && all(u(1, :) <= u_max(1)) && ...
   all(u(2, :) >= u_min(2)) && all(u(2, :) <= u_max(2))
    fprintf('All input constraints satisfied.\n');
else
    fprintf('Warning: Input constraints violated.\n');
end
