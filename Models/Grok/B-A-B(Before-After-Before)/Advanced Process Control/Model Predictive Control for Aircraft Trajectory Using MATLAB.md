% Aircraft Trajectory Optimization using MPC
% Aircraft parameters
m = 50000; % Mass (kg)
g = 9.81; % Gravitational acceleration (m/s^2)
C_D = 0.02; % Drag coefficient
C_L = 0.5; % Lift coefficient
rho = 1.225; % Air density (kg/m^3)
S = 100; % Wing area (m^2)
dt = 1; % Time step (s)
N_sim = 200; % Simulation steps

% State-space model: [x, y, vx, vy]' (position and velocity)
A = [1 0 dt 0;
     0 1 0 dt;
     0 0 1 0;
     0 0 0 1];
B = [0 0;
     0 0;
     dt/m 0;
     0 dt/m];
C = eye(4);
D = zeros(4, 2);
sys_c = ss(A, B, C, D);
sys_d = c2d(sys_c, dt, 'zoh');
Ad = sys_d.A;
Bd = sys_d.B;
Cd = sys_d.C;

% MPC Parameters
Np = 10; % Prediction horizon
Nc = 3; % Control horizon
Q = diag([10, 10, 1, 1]); % State weighting (prioritize position)
R = 0.1 * eye(2); % Input weighting (thrust, pitch)
x0 = [0; 0; 100; 10]; % Initial state: [x=0, y=0, vx=100, vy=10]
u0 = [m*g; 0]; % Initial inputs: [thrust, pitch]
u_min = [0; -0.1]; % Min thrust, pitch (rad)
u_max = [2*m*g; 0.1]; % Max thrust, pitch (rad)
y_min = 100; % Minimum altitude (m)
y_max = 10000; % Maximum altitude (m)
v_max = 200; % Maximum speed (m/s)

% Reference trajectory (straight climb with constant speed)
t = (0:N_sim-1)*dt;
x_ref = 100 * t;
y_ref = 5000 * (1 - exp(-t/50));
vx_ref = 100 * ones(1, N_sim);
vy_ref = 100 * (1 - exp(-t/50)) ./ (t + 1);
ref = [x_ref; y_ref; vx_ref; vy_ref];

% External disturbance (wind gust)
wind = @(t) [10*sin(0.1*t); 5*cos(0.2*t)]; % Wind velocity (m/s)

% Simulation storage
x = zeros(4, N_sim);
u = zeros(2, N_sim);
x(:, 1) = x0;
u(:, 1) = u0;
cost = zeros(1, N_sim);
fuel = zeros(1, N_sim); % Fuel consumption (approximated)

% MPC optimization function
function [u_opt, J] = mpc_optimize(xk, Ad, Bd, Cd, Np, Nc, Q, R, u_prev, u_min, u_max, ref_k, wind_k)
    n = size(Ad, 1);
    m = size(Bd, 2);
    H = zeros(Nc*m, Nc*m);
    f = zeros(Nc*m, 1);
    A_pred = zeros(Np*n, Nc*m);
    b_pred = zeros(Np*n, 1);
    
    % Build prediction matrices
    Phi = zeros(Np*n, n);
    Theta = zeros(Np*n, Nc*m);
    Phi(1:n, :) = Ad;
    for i = 2:Np
        Phi(i*n-(n-1):i*n, :) = Phi((i-1)*n-(n-1):(i-1)*n, :) * Ad;
    end
    
    for i = 1:Np
        for j = 1:min(i, Nc)
            if i-j >= 0
                Theta(i*n-(n-1):i*n, j*m-(m-1):j*m) = Phi((i-j)*n-(n-1):(i-j)*n, :) * Bd;
            end
        end
    end
    
47  % Cost function
    for i = 1:Np
        H = H + Theta(i*n-(n-1):i*n, :)' * Q * Theta(i*n-(n-1):i*n, :);
        f = f + Theta(i*n-(n-1):i*n, :)' * Q * (Phi(i*n-(n-1):i*n, :) * xk - ref_k(:, i));
    end
    H = H + kron(eye(Nc), R);
    
    % Constraints
    A_ineq = [kron(eye(Nc), eye(m)); -kron(eye(Nc), eye(m))];
    b_ineq = [repmat(u_max, Nc, 1); -repmat(u_min, Nc, 1)];
    
    % State constraints (altitude and speed)
    A_state = zeros(2*Np, Nc*m);
    b_state = zeros(2*Np, 1);
    for i = 1:Np
        A_state(i, :) = Cd(2, :) * Theta(i*n-(n-1):i*n, :); % Altitude
        b_state(i) = y_max - Cd(2, :) * Phi(i*n-(n-1):i*n, :) * xk;
        A_state(Np+i, :) = -Cd(2, :) * Theta(i*n-(n-1):i*n, :);
        b_state(Np+i) = -(y_min - Cd(2, :) * Phi(i*n-(n-1):i*n, :) * xk);
    end
    
    A_ineq = [A_ineq; A_state];
    b_ineq = [b_ineq; b_state];
    
    % Quadratic programming
    H = (H + H') / 2; % Ensure symmetry
    options = optimoptions('quadprog', 'Display', 'off');
    du = quadprog(H, f, A_ineq, b_ineq, [], [], [], [], [], options);
    u_opt = u_prev + du(1:m);
    J = 0.5 * du' * H * du + f' * du;
end

% Main simulation loop
for k = 1:N_sim-1
    % Current reference
    ref_k = ref(:, k:k+Np-1);
    if size(ref_k, 2) < Np
        ref_k = [ref_k, repmat(ref(:, end), 1, Np-size(ref_k, 2))];
    end
    
    % Apply wind disturbance
    w = wind(k*dt);
    Bd_w = Bd;
    Bd_w(3:4, :) = Bd_w(3:4, :) + [w(1)/m; w(2)/m]*dt;
    
    % Compute MPC control action
    [u_opt, J] = mpc_optimize(x(:, k), Ad, Bd_w, Cd, Np, Nc, Q, R, u(:, k), u_min, u_max, ref_k, w);
    u(:, k+1) = u_opt;
    cost(k) = J;
    
    % Update state with disturbance
    x(:, k+1) = Ad * x(:, k) + Bd_w * u(:, k+1);
    
    % Fuel consumption (approximated as proportional to thrust)
    fuel(k) = u(1, k) * dt / 1e6; % Scaled for readability
end
fuel(N_sim) = u(1, N_sim) * dt / 1e6;

% Compute performance metrics
error = sqrt(sum((x - ref).^2, 1));
speed = sqrt(x(3, :).^2 + x(4, :).^2);
total_fuel = sum(fuel);

% Output results
fprintf('Final Position: [%.2f, %.2f] m (Reference: [%.2f, %.2f] m)\n', ...
    x(1, end), x(2, end), ref(1, end), ref(2, end));
fprintf('Final Speed: %.2f m/s (Reference: %.2f m/s)\n', ...
    speed(end), sqrt(ref(3, end)^2 + ref(4, end)^2));
fprintf('Mean Tracking Error: %.2f m\n', mean(error));
fprintf('Total Fuel Consumption: %.2f units\n', total_fuel);
fprintf('Average Control Cost: %.2f\n', mean(cost));

% Check constraints
if all(x(2, :) >= y_min) && all(x(2, :) <= y_max) && all(speed <= v_max) && ...
   all(u(1, :) >= u_min(1)) && all(u(1, :) <= u_max(1)) && ...
   all(u(2, :) >= u_min(2)) && all(u(2, :) <= u_max(2))
    fprintf('All constraints satisfied.\n');
else
    fprintf('Warning: Constraints violated.\n');
end

% Visualization (for demonstration, not requested)
figure;
subplot(2, 1, 1);
plot(t, x(1, :), 'b-', t, ref(1, :), 'r--');
xlabel('Time (s)');
ylabel('X Position (m)');
legend('Actual', 'Reference');
grid on;
title('X Position Tracking');

subplot(2, 1, 2);
plot(t, x(2, :), 'b-', t, ref(2, :), 'r--');
xlabel('Time (s)');
ylabel('Y Position (m)');
legend('Actual', 'Reference');
grid on;
title('Y Position Tracking');
