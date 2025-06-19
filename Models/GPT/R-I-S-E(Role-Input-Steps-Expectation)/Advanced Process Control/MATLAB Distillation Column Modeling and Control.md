% System state: x = [Temperature; Composition]
% Control input: u = [Feed Rate]
% Disturbance: d = [Ambient Temp]

A = [-0.05, 0; 
      0.01, -0.03];
B = [0.8; 
     0.5];
Bd = [0.1; 
      0.02];  % disturbance gain (optional)
C = eye(2);
D = zeros(2, 1);

% Discretization
Ts = 1; % Sample time (1 min)
sys = ss(A, B, C, D);
sysd = c2d(sys, Ts);

Ad = sysd.A;
Bd = sysd.B;
Cd = sysd.C;
Dd = sysd.D;

N = 10;                 % Prediction horizon
nx = size(Ad,1);        % Number of states
nu = size(Bd,2);        % Number of inputs
x0 = [100; 0.95];       % Initial state: temp, composition

% Constraints
umin = 0.2; umax = 1.5;               % Feed rate limits
ymin = [90; 0.9]; ymax = [120; 0.98]; % Output constraints

% Setpoint
r = [110; 0.96];         % Desired temp and composition

% MPC weight matrices
Q = eye(nx);             % Output tracking weight
R = 0.1 * eye(nu);       % Input penalty

% Simulation setup
Tsim = 30;
X = zeros(nx, Tsim+1);
U = zeros(nu, Tsim);
Y = zeros(nx, Tsim);
X(:,1) = x0;

for k = 1:Tsim
    % Build optimization problem for control horizon
    H = blkdiag(kron(eye(N), Q), kron(eye(N), R));
    
    % Construct equality constraints for dynamics
    Aeq = zeros(N*nx, N*(nx+nu));
    beq = zeros(N*nx, 1);
    for i = 1:N
        Aeq((i-1)*nx+1:i*nx, (i-1)*(nx+nu)+1:(i-1)*(nx+nu)+nx) = -eye(nx);
        if i == 1
            Aeq((i-1)*nx+1:i*nx, end-nx-nu+1:end-nu) = Ad;
            Aeq((i-1)*nx+1:i*nx, end-nu+1:end) = Bd;
            beq((i-1)*nx+1:i*nx) = Ad*X(:,k) + Bd*U(:,max(k-1,1));
        else
            Aeq((i-1)*nx+1:i*nx, (i-2)*(nx+nu)+1:(i-2)*(nx+nu)+nx) = Ad;
            Aeq((i-1)*nx+1:i*nx, (i-2)*(nx+nu)+nx+1:(i-2)*(nx+nu)+nx+nu) = Bd;
        end
    end

    % Bounds
    lb = repmat([ymin; umin], N, 1);
    ub = repmat([ymax; umax], N, 1);
    
    % Cost function (simple tracking)
    ref = repmat(r, N, 1);
    f = -2 * H * [ref; zeros(N*nu,1)];
    
    % Solve QP
    options = optimoptions('quadprog', 'Display', 'none');
    [z, ~] = quadprog(2*H, f, [], [], Aeq, beq, lb, ub, [], options);
    
    % Extract control action
    u = z(N*nx+1:N*nx+nu);
    U(:,k) = u;
    
    % Simulate system
    X(:,k+1) = Ad * X(:,k) + Bd * u;
    Y(:,k) = Cd * X(:,k);
end

disp('Final States (Temp & Comp):');
disp(X(:, end));

disp('Control Inputs (Feed Rate):');
disp(U');

disp('System Outputs (Temp & Comp):');
disp(Y');
