% Aircraft States: [x; y; vx; vy]
% Control Inputs: [ax_cmd; ay_cmd] (commanded accelerations)
% External disturbance: [wind_x; wind_y]

dt = 1;     % time step [s]
N = 20;     % prediction horizon
Tsim = 50;  % total simulation time

% Discrete-time dynamics matrices (linearized)
A = [1 0 dt 0;
     0 1 0 dt;
     0 0 1  0;
     0 0 0  1];

B = [0.5*dt^2 0;
     0 0.5*dt^2;
     dt 0;
     0 dt];

E = [0.5*dt^2 0;   % disturbance matrix (wind)
     0 0.5*dt^2;
     dt 0;
     0 dt];


 nx = 4;  % number of states
nu = 2;  % number of inputs

% Initial condition
x0 = [0; 0; 0; 0];   % [x, y, vx, vy]

% Target position (e.g., destination)
xref = [1000; 500; 0; 0];

% Input constraints (e.g., max acceleration)
u_max = [5; 5]; u_min = [-5; -5];

% State constraints (optional altitude/position boundaries)
x_max = [inf; inf; 50; 50]; x_min = [-inf; -inf; -50; -50];

% Weight matrices
Q = diag([1, 1, 0.1, 0.1]);   % Penalize position error more than velocity
R = 0.01 * eye(nu);           % Penalize large control effort

X = zeros(nx, Tsim+1);
U = zeros(nu, Tsim);
X(:,1) = x0;

for k = 1:Tsim
    % Predicted wind disturbance (constant here, could be time-varying)
    wind = [2; 1];  % 2 m/s wind in x, 1 m/s in y

    % Build prediction matrices
    Phi = zeros(nx*N, nx);
    Gamma = zeros(nx*N, nu*N);
    Theta = zeros(nx*N, 2*N);

    for i = 1:N
        Phi((i-1)*nx+1:i*nx,:) = A^i;
        for j = 1:i
            Gamma((i-1)*nx+1:i*nx, (j-1)*nu+1:j*nu) = A^(i-j)*B;
            Theta((i-1)*nx+1:i*nx, (j-1)*2+1:j*2) = A^(i-j)*E;
        end
    end

    % Cost function
    Qbar = kron(eye(N), Q);
    Rbar = kron(eye(N), R);
    H = Gamma'*Qbar*Gamma + Rbar;
    f = Gamma'*Qbar*(Phi*X(:,k) + Theta*repmat(wind, N, 1) - repmat(xref, N, 1));

    % Constraints
    Aineq = [eye(nu*N); -eye(nu*N)];
    bineq = [repmat(u_max, N, 1); -repmat(u_min, N, 1)];

    % Solve QP
    options = optimoptions('quadprog', 'Display', 'off');
    u_opt = quadprog(H, f, Aineq, bineq, [], [], [], [], [], options);
    u_k = u_opt(1:nu);

    % Apply control and update state
    X(:,k+1) = A*X(:,k) + B*u_k + E*wind;
    U(:,k) = u_k;
end

disp('Final position and velocity:');
disp(X(:, end));

disp('Total fuel-related cost (sum of control effort):');
disp(sum(sum(R .* (U.^2))));

disp('Final tracking error:');
disp(norm(X(:, end) - xref));
