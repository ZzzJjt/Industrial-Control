% Model Predictive Control for Aircraft Trajectory Optimization
% Author: [Your Name]
% Description: MPC considering fuel, safety constraints, and wind disturbances

clear; clc;

%% === Aircraft Model Parameters (Longitudinal Motion) ===
% States: [altitude; velocity]
% Inputs: [thrust]
% Simplified continuous-time dynamics: 
%   altitude_dot = velocity
%   velocity_dot = (thrust - drag - wind) / mass

mass = 5000;             % Aircraft mass (kg)
g = 9.81;                % Gravity (m/s^2)
Cd = 0.03;               % Drag coefficient
A = 20;                  % Wing area (m^2)
rho = 1.225;             % Air density (kg/m^3)

Ts = 1;                  % Sample time (s)
Tsim = 60;               % Total simulation time (s)
N = 10;                  % Prediction horizon

%% === Discrete-Time Linearized Model ===
% x = [altitude; velocity], u = [thrust]
A_cont = [0 1;
          0 -Cd*rho*A/(2*mass)];
B_cont = [0;
          1/mass];

sys_c = ss(A_cont, B_cont, eye(2), zeros(2,1));
sys_d = c2d(sys_c, Ts);
Ad = sys_d.A;
Bd = sys_d.B;

%% === MPC Cost Matrices and Constraints ===
Q = diag([1, 0.5]);         % Penalize altitude error and velocity deviation
R = 0.01;                   % Penalize fuel (thrust)
umin = 0; umax = 30000;     % Thrust limits (N)
x0 = [1000; 200];           % Initial altitude 1000 m, velocity 200 m/s
xref = [1500; 250];         % Target altitude/velocity

%% === Wind Disturbance (headwind -10 m/s^2 at t = 30s) ===
wind_profile = zeros(1, Tsim);
wind_profile(30:end) = -10;

%% === Logging ===
Xlog = zeros(2, Tsim+1);
Ulog = zeros(1, Tsim);
Xlog(:,1) = x0;
x = x0;

%% === MPC Loop ===
for k = 1:Tsim
    % Build augmented prediction matrices
    Phi = zeros(2*N,2);
    Gamma = zeros(2*N,N);
    for i = 1:N
        Phi((2*i-1):2*i,:) = Ad^i;
        for j = 1:i
            Gamma((2*i-1):2*i,j) = Ad^(i-j)*Bd;
        end
    end

    % Cost function
    H = Gamma' * kron(eye(N), Q) * Gamma + R * eye(N);
    f = Gamma' * kron(eye(N), Q) * (Phi*x - repmat(xref, N, 1));

    % Constraints
    Aineq = [eye(N); -eye(N)];
    bineq = [umax*ones(N,1); -umin*ones(N,1)];

    % Solve QP
    options = optimoptions('quadprog','Display','off');
    [U,~,exitflag] = quadprog(2*H,2*f,Aineq,bineq,[],[],[],[],[],options);
    if exitflag ~= 1
        warning('QP failed at step %d', k);
        U = zeros(N,1);
    end

    % Apply first control
    u = U(1);
    wind_force = wind_profile(k);  % External disturbance

    % Simulate one step with wind disturbance
    drag_force = Cd * rho * A * x(2)^2 / 2;
    x(1) = x(1) + Ts * x(2);
    x(2) = x(2) + Ts * ((u - drag_force + wind_force)/mass);

    % Log
    Xlog(:,k+1) = x;
    Ulog(k) = u;

    % Print status
    fprintf('T=%02ds | Alt: %.1fm | Vel: %.1fm/s | Thrust: %.1fN | Wind: %.1fm/s^2\n', ...
        k, x(1), x(2), u, wind_force);
end
