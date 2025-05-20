% Aircraft Trajectory Optimization using Model Predictive Control (MPC)
% Implements MPC to optimize aircraft trajectory with fuel efficiency and disturbance compensation
% Outputs numerical results, no plots

%% Step 1: Define Aircraft Parameters and Simulation Settings
% Aircraft parameters
m = 50000; % Mass (kg)
g = 9.81; % Gravity (m/s^2)
C_D = 0.02; % Drag coefficient
C_L = 0.5; % Lift coefficient
rho = 1.225; % Air density (kg/m^3)
S = 100; % Wing area (m^2)
T_max = 200000; % Maximum thrust (N)
v_ref = 200; % Nominal airspeed (m/s)

% Simulation parameters
dt = 1; % Time step (s)
t_sim = 0:dt:200; % Simulation time (s)
N = length(t_sim);

%% Step 2: Aircraft Dynamic Model (State-Space)
% States: x = [x_pos, y_pos, v, theta] (position x, y, airspeed, flight path angle)
% Inputs: u = [T, alpha] (thrust, angle of attack)
% Linearized model around nominal condition (v = v_ref, theta = 0)
A = [0, 0, 1, 0;
     0, 0, 0, v_ref;
     0, 0, -rho*v_ref*S*C_D/m, 0;
     0, 0, 0, -rho*v_ref*S*C_L/(m*v_ref)];
B = [0, 0;
     0, 0;
     1/m, 0;
     0, rho*v_ref^2*S*C_L/(m*v_ref)];
C = eye(4); % Observe all states
D = zeros(4, 2);

% Discretize model
sys_c = ss(A, B, C, D);
sys_d = c2d(sys_c, dt, 'zoh');
Ad = sys_d.A;
Bd = sys_d.B;
Cd = sys_d.C;

%% Step 3: Define MPC Parameters and Constraints
% MPC parameters
Np = 10; % Prediction horizon
Nc = 3; % Control horizon
Q = diag([10, 10, 1, 1]); % State weights (prioritize position)
R = diag([0.01, 0.1]); % Input weights (penalize thrust for fuel efficiency)

% Constraints
T_min = 0; % Min thrust (N)
T_max = T_max; % Max thrust (N)
alpha_min = -5 * pi/180; % Min angle of attack (rad)
alpha_max = 5 * pi/180; % Max angle of attack (rad)
v_min = 150; % Min airspeed (m/s)
v_max = 250; % Max airspeed (m/s)
theta_min = -10 * pi/180; % Min flight path angle (rad)
theta_max = 10 * pi/180; % Max flight path angle (rad)
du_max = [5000; 1 * pi/180]; % Max input rate of change

% Reference trajectory (straight climb to 5000 m)
x_ref = zeros(4, N);
x_ref(1, :) = v_ref * t_sim; % x-position
x_ref(2, :) = linspace(0, 5000, N); % y-position (altitude)
x_ref(3, :) = v_ref; % Constant airspeed
x_ref(4, :) = 0; % Level flight

%% Step 4: MPC Formulation
% Augmented state-space for tracking
n = size(Ad, 1);
m = size(Bd, 2);
p = size(Cd, 1);
A_aug = [Ad, zeros(n, p); Cd*Ad, eye(p)];
B_aug = [Bd; Cd*Bd];
C_aug = [Cd, zeros(p, p)];

% Hessian and gradient
H = zeros(Nc*m, Nc*m);
for i = 1:Np
    Phi = C_aug * A_aug^i * B_aug;
    H = H + Phi' * Q * Phi;
end
H = H + kron(eye(Nc), R);

% Constraint matrices
A_con = [];
b_con = [];
for i = 1:Nc
    % Input constraints
    A_con = blkdiag(A_con, [eye(m); -eye(m)]);
    b_con = [b_con; [T_max; alpha_max]; -[T_min; alpha_min]];
    % Rate constraints
    if i == 1
        A_con = blkdiag(A_con, [eye(m); -eye(m)]);
        b_con = [b_con; du_max; du_max];
    else
        A_con = blkdiag(A_con, [eye(m), -eye(m)] * [eye(m); -eye(m)]);
        b_con = [b_con; du_max; du_max];
    end
end
% State constraints (airspeed and flight path angle)
A_state = zeros(2*Np, Nc*m);
b_state = zeros(2*Np, 1);
for i = 1:Np
    Psi = C_aug * A_aug^i * B_aug;
    A_state(2*i-1:2*i, :) = [Psi(3, :); Psi(4, :)];
    b_state(2*i-1:2*i) = [v_max; theta_max];
    A_state = [A_state; -[Psi(3, :); Psi(4, :)]];
    b_state = [b_state; -[v_min; theta_min]];
end
A_con = [A_con; A_state];
b_con = [b_con; b_state];

%% Step 5: Simulation Loop
% Initialize states and variables
x = [0; 0; v_ref; 0]; % Initial state
u = [0.5*T_max; 0]; % Initial input
x_aug = [x - x_ref(:, 1); Cd * (x - x_ref(:, 1))];
x_history = zeros(4, N);
u_history = zeros(2, N);
x_history(:, 1) = x;
u_history(:, 1) = u;

% Wind disturbance (crosswind varying sinusoidally)
wind = @(t) [10 * sin(0.05 * t); 0; 0; 0]; % Wind in x-direction (m/s)

% Simulation
for k = 1:N-1
    % Compute gradient
    f = zeros(Nc*m, 1);
    for i = 1:Np
        Phi = C_aug * A_aug^i * B_aug;
        x_pred = C_aug * A_aug^i * x_aug + x_ref(:, min(k+i, N));
        e = x_pred - x_ref(:, min(k+i, N));
        f = f + Phi' * Q * e;
    end
    
    % Solve QP problem
    options = optimoptions('quadprog', 'Display', 'off');
    delta_u = quadprog(2*H, f, A_con, b_con, [], [], [], [], [], options);
    
    % Apply first control move
    u = u + delta_u(1:m);
    u = min(max(u, [T_min; alpha_min]), [T_max; alpha_max]);
    u_history(:, k+1) = u;
    
    % Update system with disturbance
    x = Ad * x + Bd * u + wind(t_sim(k));
    x(3) = min(max(x(3), v_min), v_max); % Enforce airspeed constraints
    x(4) = min(max(x(4), theta_min), theta_max); % Enforce angle constraints
    x_history(:, k+1) = x;
    x_aug = [x - x_ref(:, k+1); Cd * (x - x_ref(:, k+1))];
end

%% Step 6: Output Results
% Print simulation results
fprintf('Simulation Results:\n');
fprintf('Time (s) | X-Pos (m) | Y-Pos (m) | Airspeed (m/s) | Theta (deg) | Thrust (kN) | Alpha (deg)\n');
fprintf('--------------------------------------------------------------------------------\n');
for k = 1:20:N
    fprintf('%.0f     | %.0f      | %.0f      | %.1f         | %.1f       | %.1f       | %.1f\n', ...
        t_sim(k), x_history(1,k), x_history(2,k), x_history(3,k), ...
        x_history(4,k)*180/pi, u_history(1,k)/1000, u_history(2,k)*180/pi);
end

% Performance metrics
error = x_history - x_ref(:, 1:N);
mse = mean(sum(error.^2, 1));
settling_idx = find(abs(error(2,:)) < 50, 1, 'first'); % Altitude within 50 m
settling_time = t_sim(settling_idx) * (settling_idx <= N);
fuel_usage = sum(u_history(1,:)) * dt / 3600; % Approximate fuel (thrust-time)
fprintf('\nPerformance Metrics:\n');
fprintf('Mean Squared Error: %.2f m^2\n', mse);
fprintf('Settling Time (altitude): %.0f s\n', settling_time);
fprintf('Fuel Usage (thrust-time): %.2f kN-h\n', fuel_usage);
