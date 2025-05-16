% Distillation Column MPC Control
% Models a 10-tray binary distillation column and implements MPC
% No plotting or visualization included

% --- Reactor Model ---
function dx = column_dynamics(t, x, u, F_disturb)
    % States: x = [T1, T2, ..., T10] (tray temperatures, °C)
    % Inputs: u = [F, R, Q] (feed rate kmol/s, reflux rate kmol/s, reboiler duty kW)
    % Disturbance: F_disturb (additional feed flow rate, kmol/s)
    
    % Parameters
    N = 10; % Number of trays
    alpha = 2.5; % Relative volatility (methanol-water)
    V = 1.5; % Vapor flow rate (kmol/s, nominal)
    L = u(2); % Liquid flow (reflux rate)
    F = u(1) + F_disturb; % Total feed rate
    Q = u(3); % Reboiler duty
    T_feed = 70.0; % Feed temperature (°C)
    x_F = 0.5; % Feed methanol mole fraction
    Cp = 75.0; % Heat capacity (kJ/kmol/°C)
    Hv = 35000; % Heat of vaporization (kJ/kmol)
    
    dx = zeros(N, 1);
    % Tray dynamics: Energy balance with nonlinear VLE
    for i = 1:N
        % Vapor-liquid equilibrium (simplified)
        if i == 1
            y_i = x_F * alpha / (1 + (alpha - 1) * x_F); % Feed tray approximation
        else
            y_i = x(i) * alpha / (1 + (alpha - 1) * x(i));
        end
        % Liquid flow from tray above
        if i == 1
            L_in = L;
        else
            L_in = L;
        end
        % Vapor flow from tray below
        if i == N
            V_in = Q / Hv; % Reboiler vapor
        else
            V_in = V;
        end
        % Energy balance: dT/dt = (L_in*T_above + V_in*T_below - L*T_i - V*T_i) / (M*Cp)
        M = 10.0; % Tray holdup (kmol)
        if i == 1
            T_above = T_feed; % Feed temperature
        else
            T_above = x(i-1);
        end
        if i == N
            T_below = x(i); % Reboiler assumption
        else
            T_below = x(i+1);
        end
        dT_dt = (L_in * T_above + V_in * T_below - L * x(i) - V * x(i)) / (M * Cp);
        dx(i) = dT_dt;
    end
end

% --- Linearized Model for MPC ---
function [A, B, C, D] = linearize_model(x0, u0)
    % Nominal operating condition
    % x0: Steady-state temperatures (T1=65°C, T10=95°C, interpolated)
    % u0: [F=1, R=1, Q=3000]
    N = 10;
    A = zeros(N, N);
    B = zeros(N, 3);
    C = zeros(2, N);
    D = zeros(2, 3);
    
    % Simplified linear model (finite difference approximation)
    M = 10.0; % Tray holdup (kmol)
    Cp = 75.0; % Heat capacity (kJ/kmol/°C)
    Hv = 35000; % Heat of vaporization (kJ/kmol)
    V = 1.5; % Vapor flow (kmol/s)
    L = u0(2); % Reflux rate
    
    for i = 1:N
        A(i, i) = -(L + V) / (M * Cp); % Diagonal: self-dynamics
        if i > 1
            A(i, i-1) = L / (M * Cp); % Effect of tray above
        end
        if i < N
            A(i, i+1) = V / (M * Cp); % Effect of tray below
        end
    end
    % Input effects
    B(:, 1) = 0.1 / (M * Cp); % Feed rate (F)
    B(:, 2) = 0.2 / (M * Cp); % Reflux rate (R)
    B(N, 3) = 1 / (M * Hv); % Reboiler duty (Q)
    % Outputs: T1 and T10
    C(1, 1) = 1; % T1
    C(2, N) = 1; % T10
end

% --- MPC Setup ---
% Nominal operating condition
x0 = linspace(65, 95, 10)'; % Steady-state temperatures (°C)
u0 = [1.0; 1.0; 3000]; % F=1 kmol/s, R=1 kmol/s, Q=3000 kW
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
mpcobj.MV(1).Min = 0.8; % F (kmol/s)
mpcobj.MV(1).Max = 1.5;
mpcobj.MV(2).Min = 0.5; % R (kmol/s)
mpcobj.MV(2).Max = 2.0;
mpcobj.MV(3).Min = 0; % Q (kW)
mpcobj.MV(3).Max = 5000;
mpcobj.MV(1).RateMin = -0.1; % Rate limits
mpcobj.MV(1).RateMax = 0.1;
mpcobj.MV(2).RateMin = -0.1;
mpcobj.MV(2).RateMax = 0.1;
mpcobj.MV(3).RateMin = -100;
mpcobj.MV(3).RateMax = 100;
mpcobj.OV(1).Min = 60; % T1 (°C)
mpcobj.OV(1).Max = 100;
mpcobj.OV(2).Min = 60; % T10 (°C)
mpcobj.OV(2).Max = 100;

% Weights
mpcobj.Weights.OV = [1 1]; % High weight on T1, T10 errors
mpcobj.Weights.MV = [0.1 0.1 0.01]; % Low weight on control effort
mpcobj.Weights.MVrate = [0.1 0.1 0.01]; % Penalize rapid changes

% --- Simulation ---
t_sim = 1000; % Simulation time (s)
t = 0:dt:t_sim;
n_steps = length(t);
x = x0; % Initial state
u = u0; % Initial input
y = C * x; % Initial output
u_history = zeros(3, n_steps);
x_history = zeros(10, n_steps);
y_history = zeros(2, n_steps);

% Setpoints
SP = [65; 95]; % T1=65°C, T10=95°C
r = repmat(SP, 1, n_steps);

% Disturbance: Feed flow rate step at t=200s
F_disturb = zeros(1, n_steps);
F_disturb(t >= 200) = 0.2; % Step from 1 to 1.2 kmol/s

% MPC simulation
for k = 1:n_steps
    % Store current state
    x_history(:, k) = x;
    y_history(:, k) = y;
    
    % Compute MPC control action
    u = mpcmove(mpcobj, x, y, r(:, k), []);
    u_history(:, k) = u;
    
    % Simulate nonlinear model
    [~, x_new] = ode45(@(t, x) column_dynamics(t, x, u, F_disturb(k)), [0 dt], x);
    x = x_new(end, :)';
    y = C * x;
end

% --- End of Simulation ---
% Note: No plotting or visualization included as per requirements
% Outputs (x_history, y_history, u_history) can be used for analysis
