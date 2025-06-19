function dxdt = aircraft_dynamics(t, state, u, params)
    % State vector: [x; y; v; theta]
    x = state(1);
    y = state(2);
    v = state(3);
    theta = state(4);

    % Inputs: [T; alpha]
    T = u(1);
    alpha = u(2);

    % Parameters
    m = params.m;
    g = params.g;
    CL = params.CL;
    CD = params.CD;
    rho = params.rho;
    S = params.S;

    % Wind disturbance
    wx = params.wind_x(t);   % Function handle for time-varying wind
    wy = params.wind_y(t);

    % Aerodynamic forces
    q = 0.5 * rho * v^2;
    L = q * S * CL;
    D = q * S * CD;

    % Dynamics
    dxdt = [
        v * cos(theta) + wx;
        v * sin(theta) + wy;
        (T * cos(alpha) - D) / m;
        (T * sin(alpha) + L) / (m * v)
    ];
end

% Linearized aircraft model (simplified for MPC design)
A = eye(4);  % Placeholder – should be linearized around operating point
B = [0 0; 0 0; 1/m 0; 0 1/(m*v)];
C = eye(4);
D = zeros(4, 2);

sys = ss(A, B, C, D, 'Ts', 0);  % Continuous-time model

% Discretize
Ts = 1;
sysd = c2d(sys, Ts);

% Create MPC object
mpcobj = mpc(sysd, Ts, 20, 5);  % Prediction horizon = 20, Control horizon = 5

% Set weights
mpcobj.Weights.OV = [10, 10, 1, 1];  % Track x, y tightly
mpcobj.Weights.MV = [0.1, 0.01];     % Penalize thrust and AoA changes

% Input constraints (Thrust, AoA)
mpcobj.MV(1).Min = 0;       % Thrust min
mpcobj.MV(1).Max = 50000;   % Thrust max
mpcobj.MV(2).Min = -10*pi/180;  % AoA min (-10 deg)
mpcobj.MV(2).Max = 15*pi/180;   % AoA max (15 deg)

% Output constraints (Position, Velocity, Angle)
mpcobj.OV(1).Min = -inf; mpcobj.OV(1).Max = inf;  % x
mpcobj.OV(2).Min = 1000; mpcobj.OV(2).Max = 10000;  % y (altitude)
mpcobj.OV(3).Min = 100; mpcobj.OV(3).Max = 250;     % v
mpcobj.OV(4).Min = -pi/4; mpcobj.OV(4).Max = pi/4;  % theta

% Simulation setup
Tstop = 600;  % seconds
time = 0:Ts:Tstop;
N = length(time);

% Initial state: [x; y; v; theta]
x0 = [0; 2000; 200; 0];

% Reference trajectory (straight level flight at 200 m/s, 3000 m)
ref = repmat([10000; 3000; 200; 0], 1, N)';
ref(:, 2) = 3000 * ones(N, 1);  % Maintain altitude

% Initialize simulation
x = x0;
log_states = zeros(N, 4);
log_inputs = zeros(N, 2);

for k = 1:N
    % Update reference window
    ref_k = ref(k:min(k+mpcobj.PredictionHorizon, N), :);
    
    % Compute optimal control move
    u = mpcmove(mpcobj, x, ref_k, []);
    
    % Apply input and simulate dynamics
    params = struct('m', 10000, 'g', 9.81, ...
                    'CL', 0.5, 'CD', 0.02, ...
                    'rho', 1.225, 'S', 30, ...
                    'wind_x', @(t) 10*sin(t/60), ...  % Time-varying wind
                    'wind_y', @(t) 2*cos(t/120));

    % Integrate dynamics forward (e.g., using ode45)
    tspan = [time(k), time(k)+Ts];
    [t_sim, x_traj] = ode45(@(t, s) aircraft_dynamics(t, s, u, params), tspan, x);
    x = x_traj(end, :)';

    % Log results
    log_states(k, :) = x;
    log_inputs(k, :) = u;
end

% Fuel consumption (approximate as integral of thrust)
fuel_consumed = sum(log_inputs(:, 1) * Ts) / 1e6;  % MJ or arbitrary unit

% Tracking error
tracking_error = mean(sqrt((log_states(:, 1) - ref(:, 1)).^2 + ...
                           (log_states(:, 2) - ref(:, 2)).^2));

fprintf('Total Fuel Consumed: %.2f units\n', fuel_consumed);
fprintf('Average Tracking Error: %.2f meters\n', tracking_error);
