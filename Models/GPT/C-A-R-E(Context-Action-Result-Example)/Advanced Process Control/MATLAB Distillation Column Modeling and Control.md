% Distillation Column Model and MPC Implementation (No Plotting)
clear; clc;

% Simulation parameters
T_final = 100;           % total simulation time steps
dt = 1;                  % sampling time
n_states = 2;            % [TopTemperature; BottomTemperature]
n_inputs = 1;            % [RefluxRatio]

% System initial conditions
x = [80; 120];           % Initial top and bottom temperatures
u = 0.5;                 % Initial reflux ratio
feed_rate = 1.0;         % Initial feed rate
x_log = zeros(n_states, T_final);
u_log = zeros(n_inputs, T_final);

% MPC setup
Np = 10;                 % Prediction horizon
Nc = 3;                  % Control horizon
u_min = 0.1;
u_max = 1.0;

% Desired setpoint
x_sp = [78; 122];

% Weighting matrices
Q = eye(n_states);
R = 0.1;

% Model: dx/dt = f(x,u,feed)
model = @(x,u,feed) [ ...
    -0.2*(x(1) - 78) + 0.1*u - 0.05*feed;   % Top tray dynamics
    -0.3*(x(2) - 120) + 0.1*(1 - u) + 0.07*feed];  % Bottom tray dynamics

% Main simulation loop
for k = 1:T_final
    % Disturbance: step change in feed rate at t=50
    if k == 50
        feed_rate = 1.2;
    end

    % Linearization (Jacobian) around current state
    A = [-0.2, 0; 0, -0.3];
    B = [0.1; -0.1];
    Bd = [-0.05; 0.07];

    % Discretize
    Ad = eye(n_states) + A*dt;
    Bd = B*dt;
    Dd = Bd * feed_rate;

    % Build prediction model
    Phi = zeros(Np*n_states, n_states);
    Gamma = zeros(Np*n_states, Nc);
    C = eye(n_states);

    for i = 1:Np
        Phi((i-1)*n_states+1:i*n_states, :) = Ad^i;
        for j = 1:Nc
            if i >= j
                Gamma_block = Ad^(i-j)*Bd;
                Gamma((i-1)*n_states+1:i*n_states, j) = Gamma_block;
            end
        end
    end

    % Setup MPC QP problem
    H = Gamma' * kron(eye(Np), Q) * Gamma + R * eye(Nc);
    f = Gamma' * kron(eye(Np), Q) * (Phi*x - repmat(x_sp, Np, 1));

    % Solve QP
    du = quadprog(H, f, [], [], [], [], ...
                  u_min - u, u_max - u, [], optimset('Display','off'));

    % Apply first control move
    u = u + du(1);
    u = min(max(u, u_min), u_max);

    % Update process with nonlinear model
    dx = model(x, u, feed_rate);
    x = x + dx * dt;

    % Log data
    x_log(:,k) = x;
    u_log(:,k) = u;
end
