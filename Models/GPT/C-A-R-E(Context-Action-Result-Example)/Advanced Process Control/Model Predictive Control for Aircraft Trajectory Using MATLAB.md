clear; clc;

% Time settings
T = 50;           % Simulation time steps
dt = 1.0;         % Time step (s)
Np = 10;          % Prediction horizon

% State: [x; z; vx; vz] → position and velocity in x and z
n_states = 4;
n_inputs = 2;     % [ax; az] acceleration input

% Initial state
x0 = [0; 1000; 100; 0];  % start at 0 m, 1000 m altitude, 100 m/s horiz, 0 m/s vertical
x = x0;
X_log = zeros(n_states, T);
U_log = zeros(n_inputs, T);

% Target
target = [5000; 1200];  % desired position (x, z)

% Weights
Q = diag([1, 5, 0.1, 0.1]);       % tracking + velocity shaping
R = 0.01 * eye(n_inputs);        % fuel efficiency (penalize large thrust)

% Constraints
a_max = 5;     % max acceleration (m/s^2)

% External disturbance: wind gust at t = 25
wind_profile = zeros(2, T);  % [dx_wind; dz_wind]
wind_profile(:, 25:30) = repmat([10; 2], 1, 6);  % wind adds 10 m/s horiz, 2 m/s vertical

% Prediction matrices
A = [1 0 dt 0;
     0 1 0 dt;
     0 0 1 0;
     0 0 0 1];
B = [0 0;
     0 0;
     dt 0;
     0 dt];

for k = 1:T
    % Reference trajectory: linearly approach target
    ref = repmat([target; 0; 0], Np, 1);

    % Build prediction matrices
    Phi = zeros(Np*n_states, n_states);
    Gamma = zeros(Np*n_states, Np*n_inputs);
    for i = 1:Np
        Phi((i-1)*n_states+1:i*n_states,:) = A^i;
        for j = 1:i
            Gamma((i-1)*n_states+1:i*n_states,(j-1)*n_inputs+1:j*n_inputs) = A^(i-j)*B;
        end
    end

    % Build cost function
    H = Gamma' * kron(eye(Np), Q) * Gamma + kron(eye(Np), R);
    f = Gamma' * kron(eye(Np), Q) * (Phi*x - ref);

    % Constraints: acceleration limits
    A_con = [eye(Np*n_inputs); -eye(Np*n_inputs)];
    b_con = [a_max*ones(Np*n_inputs,1); a_max*ones(Np*n_inputs,1)];

    % Solve QP
    options = optimoptions('quadprog', 'Display', 'off');
    u_seq = quadprog(H, f, A_con, b_con, [], [], [], [], [], options);

    % Apply first control input
    u = u_seq(1:n_inputs);
    U_log(:,k) = u;

    % Add wind disturbance
    wind = wind_profile(:,k);

    % Update state
    x = A*x + B*u + [0; 0; wind(1); wind(2)];
    X_log(:,k) = x;
end
