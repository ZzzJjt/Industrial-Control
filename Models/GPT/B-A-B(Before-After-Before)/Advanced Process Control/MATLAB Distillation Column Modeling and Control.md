% ----------------------------
% Distillation Column Modeling
% ----------------------------

% Discretization parameters
Ts = 1.0;  % Sampling time (seconds)
N_sim = 60;  % Simulation steps

% State-space model (simplified)
% States: [Tray Temp 1; Tray Temp 2]
% Input: Feed rate
A = [0.9 0.1; 0.05 0.95];
B = [0.05; 0.1];
C = eye(2);
D = [0; 0];

% Define constraints
umin = 0;    % min feed rate
umax = 10;   % max feed rate
Tmin = 70;   % min temperature (°C)
Tmax = 120;  % max temperature (°C)

% Initial conditions
x = [80; 85];      % Initial tray temperatures
xref = [100; 105]; % Desired tray temperatures
u = 5;             % Initial feed rate

% MPC parameters
Np = 10;  % Prediction horizon
Nc = 3;   % Control horizon
Q = eye(2);       % Output weighting
R = 0.1;          % Input weighting

% ----------------------------
% MPC Control Loop
% ----------------------------
X_log = zeros(2, N_sim);
U_log = zeros(1, N_sim);

for k = 1:N_sim
    % Prediction model setup
    H = []; F = [];   % Quadratic cost
    Aeq = []; beq = [];
    Aineq = []; bineq = [];
    
    % Build prediction matrices
    Phi = zeros(2*Np, Nc);
    Gamma = zeros(2*Np, 2);
    for i = 1:Np
        Ai = A^i;
        Gamma(2*i-1:2*i,:) = Ai;
        for j = 1:Nc
            if i >= j
                Phi(2*i-1:2*i,j) = A^(i-j)*B;
            end
        end
    end

    % Objective: min (Y - Yref)'Q(Y - Yref) + U'RU
    H = Phi'*kron(eye(Np), Q)*Phi + R*eye(Nc);
    f = Phi'*kron(eye(Np), Q)*(Gamma*x - repmat(xref, Np, 1));

    % Input constraints
    Aineq = [eye(Nc); -eye(Nc)];
    bineq = [repmat(umax - u, Nc, 1); repmat(u - umin, Nc, 1)];

    % Solve quadratic program (quadprog)
    options = optimoptions('quadprog','Display','off');
    delta_u = quadprog(H, f, Aineq, bineq, Aeq, beq, [], [], [], options);

    % Apply control input
    if isempty(delta_u)
        delta_u = zeros(Nc,1);  % fallback
    end
    u = u + delta_u(1);  % Only apply first input move

    % System update
    x = A*x + B*u;

    % Log
    X_log(:,k) = x;
    U_log(k) = u;

    % Optional: clamp state to safety bounds
    x = max(min(x, Tmax), Tmin);
end

% ----------------------------
% Output: Numerical Logs
% ----------------------------
disp('Final Tray Temperatures:');
disp(x);
disp('Final Feed Rate:');
disp(u);
disp('All Control Inputs:');
disp(U_log');
disp('All Tray Temps:');
disp(X_log');
