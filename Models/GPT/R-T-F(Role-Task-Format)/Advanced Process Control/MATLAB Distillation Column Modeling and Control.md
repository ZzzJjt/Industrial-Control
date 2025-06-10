% MPC for Distillation Column Temperature Control
% Author: [Your Name]
% Description: Simulates a distillation column's temperature dynamics and controls it using MPC without graphical output.

clear; clc;

%% System Parameters
n_states = 2;  % [TrayTemp, ReboilerTemp]
n_inputs = 1;  % [FeedRate]
n_outputs = 1; % [TrayTemp]

A = [-0.05  0.02;
      0     -0.1];   % State matrix

B = [0.05;
     0.1];           % Input matrix

C = [1 0];           % Output matrix (monitor tray temperature)
D = 0;

Ts = 1;              % Sampling time (minutes)
T_sim = 60;          % Total simulation time (minutes)
N = 10;              % Prediction horizon

%% Discretize model
sysc = ss(A, B, C, D);
sysd = c2d(sysc, Ts);

Ad = sysd.A;
Bd = sysd.B;
Cd = sysd.C;
Dd = sysd.D;

%% MPC Matrices
Q = eye(n_states);         % State weighting
R = 0.1;                   % Input weighting
umin = 0.8; umax = 1.2;    % Feed rate constraints (e.g., kg/min)
x = zeros(n_states, 1);    % Initial state
xref = [80; 90];           % Desired temp: Tray 80°C, Reboiler 90°C

%% Storage for logging
log_x = zeros(n_states, T_sim);
log_u = zeros(n_inputs, T_sim);

%% MPC loop
for k = 1:T_sim
    % Build prediction model matrices
    Phi = zeros(n_states*N, n_states);
    Gamma = zeros(n_states*N, N);
    for i = 1:N
        Phi((i-1)*n_states+1:i*n_states, :) = Ad^i;
        for j = 1:i
            Gamma((i-1)*n_states+1:i*n_states, j) = Ad^(i-j) * Bd;
        end
    end

    % Define cost function matrices
    H = Gamma' * kron(eye(N), Q) * Gamma + R * eye(N);
    f = Gamma' * kron(eye(N), Q) * (Phi * x - repmat(xref, N, 1));

    % Constraints: bounds on u
    Aineq = [eye(N); -eye(N)];
    bineq = [umax*ones(N, 1); -umin*ones(N, 1)];

    % Solve QP problem
    options = optimoptions('quadprog', 'Display', 'off');
    [U, ~] = quadprog(2*H, 2*f, Aineq, bineq, [], [], [], [], [], options);

    % Apply first control move
    u = U(1);
    x = Ad * x + Bd * u;

    % Log data
    log_x(:, k) = x;
    log_u(:, k) = u;

    % Print status
    fprintf('Time %2d min | Tray Temp: %.2f°C | Reboiler Temp: %.2f°C | FeedRate: %.2f kg/min\n', ...
        k, x(1), x(2), u);
end
