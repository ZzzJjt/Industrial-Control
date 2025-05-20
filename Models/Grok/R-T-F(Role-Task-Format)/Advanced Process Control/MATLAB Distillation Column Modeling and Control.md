% Distillation Column Model and MPC Implementation
% Models temperature dynamics based on feed rates and applies MPC
% No graphical outputs, results printed to console

%% Step 1: Define System Parameters
% System parameters
n_trays = 10; % Number of trays in the column
V = 1.0; % Vapor flow rate (kmol/min)
L = 0.8; % Liquid flow rate (kmol/min)
F = 1.0; % Feed flow rate (kmol/min, manipulated variable)
alpha = 2.0; % Relative volatility
x_F = 0.5; % Feed composition (mole fraction)
T_ref = 350; % Reference temperature (K)
Cp = 100; % Heat capacity (J/kmol-K)
deltaH_vap = 30000; % Heat of vaporization (J/kmol)

% Time parameters
dt = 0.1; % Time step (min)
t_sim = 0:dt:20; % Simulation time (min)
N = length(t_sim);

%% Step 2: Mathematical Modeling (State-Space)
% State vector: x = [T_1, T_2, ..., T_n] (tray temperatures)
% Input: u = F (feed flow rate)
% Output: y = T_n (top tray temperature)
% Linearized state-space model around operating condition
A = zeros(n_trays, n_trays);
B = zeros(n_trays, 1);
C = zeros(1, n_trays);
C(1, n_trays) = 1; % Output is top tray temperature

% Populate A matrix (tridiagonal for tray interactions)
for i = 1:n_trays
    if i == 1
        A(i, i) = -(V + L) / (Cp * V);
        A(i, i+1) = V / (Cp * V);
    elseif i == n_trays
        A(i, i-1) = L / (Cp * V);
        A(i, i) = -(V + L) / (Cp * V);
    else
        A(i, i-1) = L / (Cp * V);
        A(i, i) = -(V + L) / (Cp * V);
        A(i, i+1) = V / (Cp * V);
    end
end

% B matrix (feed affects middle tray, assume feed tray = n_trays/2)
feed_tray = round(n_trays/2);
B(feed_tray, 1) = deltaH_vap * x_F / (Cp * V);

% Discretize state-space model
sys_c = ss(A, B, C, 0);
sys_d = c2d(sys_c, dt, 'zoh');
Ad = sys_d.A;
Bd = sys_d.B;
Cd = sys_d.C;

%% Step 3: Define MPC Parameters and Constraints
% MPC parameters
Np = 10; % Prediction horizon
Nc = 3; % Control horizon
Q = 10; % Output weight
R = 0.1; % Input weight

% Constraints
u_min = 0.5; % Minimum feed rate (kmol/min)
u_max = 1.5; % Maximum feed rate (kmol/min)
y_min = T_ref - 10; % Minimum temperature (K)
y_max = T_ref + 10; % Maximum temperature (K)
du_max = 0.1; % Maximum rate of change in feed rate (kmol/min)

% Setpoint
T_sp = T_ref + 5; % Target top tray temperature (K)

%% Step 4: MPC Implementation
% Precompute MPC gain matrices
% Augmented state-space model for integral action
n = size(Ad, 1);
m = size(Bd, 2);
p = size(Cd, 1);
A_aug = [Ad, zeros(n, p); Cd*Ad, eye(p)];
B_aug = [Bd; Cd*Bd];
C_aug = [Cd, zeros(p, p)];

% Hessian and gradient for quadratic programming
H = zeros(Nc*m, Nc*m);
f = zeros(Nc*m, 1);
for i = 1:Np
    Phi = C_aug * A_aug^i * B_aug;
    H = H + Phi' * Q * Phi;
    % f will be computed in loop
end
H = H + R * eye(Nc*m);

% Constraint matrices
A_con = [];
b_con = [];
for i = 1:Nc
    % Input constraints
    A_con = blkdiag(A_con, [eye(m); -eye(m)]);
    b_con = [b_con; u_max*ones(m, 1); -u_min*ones(m, 1)];
    % Rate constraints
    if i == 1
        A_con = blkdiag(A_con, [eye(m); -eye(m)]);
        b_con = [b_con; du_max*ones(m, 1); du_max*ones(m, 1)];
    else
        A_con = blkdiag(A_con, [eye(m), -eye(m)] * [eye(m); -eye(m)]);
        b_con = [b_con; du_max*ones(m, 1); du_max*ones(m, 1)];
    end
end

%% Step 5: Simulation Loop
% Initialize states and variables
x = zeros(n_trays, 1); % Initial tray temperatures (relative to T_ref)
u = F; % Initial feed rate
y = Cd * x; % Initial output
x_aug = [x; y - T_sp]; % Augmented state
u_history = zeros(1, N);
y_history = zeros(1, N);
u_history(1) = u;
y_history(1) = y + T_ref; % Absolute temperature

% Simulation
for k = 1:N-1
    % Compute gradient for current error
    y_pred = zeros(Np, 1);
    for i = 1:Np
        y_pred(i) = C_aug * A_aug^i * x_aug;
    end
    e = y_pred - T_sp;
    f = zeros(Nc*m, 1);
    for i = 1:Np
        Phi = C_aug * A_aug^i * B_aug;
        f = f + Phi' * Q * e(i);
    end
    
    % Solve QP problem (using quadprog)
    options = optimoptions('quadprog', 'Display', 'off');
    delta_u = quadprog(2*H, f, A_con, b_con, [], [], [], [], [], options);
    
    % Apply first control move
    u = u + delta_u(1);
    u = min(max(u, u_min), u_max); % Enforce input constraints
    u_history(k+1) = u;
    
    % Update system
    x = Ad * x + Bd * (u - F); % Deviation from nominal
    y = Cd * x;
    y = min(max(y + T_ref, y_min), y_max); % Enforce output constraints
    y_history(k+1) = y;
    x_aug = [x; y - T_sp];
end

%% Step 6: Display Results
% Print key simulation results
fprintf('Simulation Results:\n');
fprintf('Time (min) | Feed Rate (kmol/min) | Top Tray Temp (K)\n');
fprintf('----------------------------------------\n');
for k = 1:10:N
    fprintf('%.1f       | %.3f              | %.2f\n', t_sim(k), u_history(k), y_history(k));
end

% Calculate and display performance metrics
error = y_history - T_sp;
mse = mean(error.^2);
settling_idx = find(abs(error) < 0.01 * T_sp, 1, 'first');
settling_time = t_sim(settling_idx) * (settling_idx <= N);
fprintf('\nPerformance Metrics:\n');
fprintf('Mean Squared Error: %.2f K^2\n', mse);
fprintf('Settling Time: %.1f min\n', settling_time);
