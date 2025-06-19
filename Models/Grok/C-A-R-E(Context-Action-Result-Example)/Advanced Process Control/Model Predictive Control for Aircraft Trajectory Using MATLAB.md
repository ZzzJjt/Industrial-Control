% Aircraft Trajectory Optimization using MPC
% Models 3DOF longitudinal dynamics and implements MPC
% No plotting or visualization included

% --- Aircraft Dynamics ---
function dx = aircraft_dynamics(t, x, u, w)
    % States: x = [x_pos, y_pos, h, v] (m, m, m, m/s)
    % Inputs: u = [T, theta] (thrust kN, pitch angle rad)
    % Disturbance: w = [wx, wy] (wind speed m/s in x, y)
    
    % Parameters
    m = 50000; % Mass (kg)
    g = 9.81; % Gravity (m/s^2)
    rho = 1.225; % Air density (kg/m^3)
    S = 100; % Wing area (m^2)
    CD = 0.02; % Drag coefficient
    CL = 1.0; % Lift coefficient (simplified)
    T = u(1) * 1000; % Thrust (N)
    theta = u(2); % Pitch angle (rad)
    wx = w(1); wy = w(2);
    
    % Nonlinear dynamics
    v = x(4); % Velocity
    % Airspeed components
    v_x = v * cos(theta) + wx;
    v_y = v * sin(theta) + wy;
    % Forces
    L = 0.5 * rho * v^2 * S * CL; % Lift
    D = 0.5 * rho * v^2 * S * CD; % Drag
    % Equations of motion
    dx_dt = v_x; % x-position
    dy_dt = v_y; % y-position
    dh_dt = v * sin(theta); % Altitude
    dv_dt = (T * cos(theta) - D - m * g * sin(theta)) / m; % Velocity
    
    dx = [dx_dt; dy_dt; dh_dt; dv_dt];
end

% --- Linearized Model for MPC ---
function [A, B, C, D] = linearize_model(x0, u0)
    % Nominal condition: x0 = [0, 0, 3000, 200], u0 = [50, 0]
    % States: x = [x, y, h, v], Inputs: u = [T, theta]
    m = 50000; % Mass (kg)
    g = 9.81; % Gravity (m/s^2)
    rho = 1.225; % Air density (kg/m^3)
    S = 100; % Wing area (m^2)
    CD = 0.02; % Drag coefficient
    CL = 1.0; % Lift coefficient
    v0 = x0(4); % Nominal velocity
    theta0 = u0(2); % Nominal pitch
    
    % Linearized dynamics
    A = zeros(4, 4);
    A(1, 4) = cos(theta0); % dx/dt = v * cos(theta)
    A(2, 4) = sin(theta0); % dy/dt = v * sin(theta)
    A(3, 4) = sin(theta0); % dh/dt = v * sin(theta)
    A(4, 4) = -rho * v0 * S * CD / m; % dv/dt = -D/m
    
    B = zeros(4, 2);
    B(4, 1) = 1000 / m; % dv/dt = T/m (T in kN)
    B(1, 2) = -v0 * sin(theta0); % dx/dt = -v * sin(theta)
    B(2, 2) = v0 * cos(theta0); % dy/dt = v * cos(theta)
    B(3, 2) = v0 * cos(theta0); % dh/dt = v * cos(theta)
    B(4, 2) = (-1000 * u0(1) * sin(theta0) - m * g * cos(theta0)) / m; % dv/dt = -T*sin(theta) - g*cos(theta)
    
    C = eye(4); % All states as outputs
    D = zeros(4, 2);
end

% --- MPC Setup ---
% Nominal condition
x0 = [0; 0; 3000; 200]; % [x, y, h, v]
u0 = [50; 0]; % [T, theta]
[A, B, C, D] = linearize_model(x0, u0);

% Discrete-time model
dt = 1.0; % Time step (s)
sys_c = ss(A, B, C, D);
sys_d = c2d(sys_c, dt);

% MPC controller
mpcobj = mpc(sys_d, dt);
mpcobj.PredictionHorizon = 10;
mpcobj.ControlHorizon = 3;

% Constraints
mpcobj.MV(1).Min = 0; % T (kN)
mpcobj.MV(1).Max = 100;
mpcobj.MV(2).Min = -0.2; % theta (rad)
mpcobj.MV(2).Max = 0.2;
mpcobj.MV(1).RateMin = -5; % Rate limits
mpcobj.MV(1).RateMax = 5;
mpcobj.MV(2).RateMin = -0.02;
mpcobj.MV(2).RateMax = 0.02;
mpcobj.OV(3).Min = 500; % h (m)
mpcobj.OV(3).Max = 10000;
mpcobj.OV(4).Min = 150; % v (m/s)
mpcobj.OV(4).Max = 250;

% Weights
mpcobj.Weights.OV = [1 1 10 5]; % High on h, v; moderate on x, y
mpcobj.Weights.MV = [0.1 0.1]; % Low on control effort
mpcobj.Weights.MVrate = [0.1 0.1]; % Penalize rapid changes

% --- Reference Trajectory ---
% Linear climb from (0, 0, 1000) to (50000, 0, 5000) over 1000s
t_sim = 1000;
t = 0:dt:t_sim;
n_steps = length(t);
x_ref = linspace(0, 50000, n_steps);
y_ref = zeros(1, n_steps);
h_ref = linspace(1000, 5000, n_steps);
v_ref = 200 * ones(1, n_steps);
r = [x_ref; y_ref; h_ref; v_ref];

% --- Disturbance: Wind Gust ---
w = zeros(2, n_steps); % [wx, wy]
w(1, t >= 200 & t < 300) = 10; % 10 m/s gust in x-direction

% --- Simulation ---
x = x0; % Initial state
u = u0; % Initial input
y = C * x; % Initial output
x_history = zeros(4, n_steps);
y_history = zeros(4, n_steps);
u_history = zeros(2, n_steps);

% MPC simulation
for k = 1:n_steps
    % Store current state
    x_history(:, k) = x;
    y_history(:, k) = y;
    
    % Compute MPC control action
    u = mpcmove(mpcobj, x, y, r(:, k), []);
    u_history(:, k) = u;
    
    % Simulate nonlinear model
    [~, x_new] = ode45(@(t, x) aircraft_dynamics(t, x, u, w(:, k)), [0 dt], x);
    x = x_new(end, :)';
    y = C * x;
end

% --- End of Simulation ---
% Note: No plotting included as per requirements
% Outputs (x_history, y_history, u_history) stored for analysis
