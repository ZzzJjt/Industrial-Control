% Aircraft Trajectory Planning using MPC
clear; clc;

% Define simulation parameters
Ts = 0.1;           % Sampling time [s]
T_sim = 20;         % Total simulation time [s]
N = 20;             % Prediction horizon
nx = 4;             % States: [x; y; vx; vy]
nu = 2;             % Inputs: [ax; ay]

% Define the state-space model
A = [1 0 Ts 0;
     0 1 0 Ts;
     0 0 1 0;
     0 0 0 1];

B = [0.5*Ts^2 0;
     0 0.5*Ts^2;
     Ts 0;
     0 Ts];

C = eye(nx);
D = zeros(nx, nu);

sys = ss(A,B,C,D,Ts);

% Create MPC controller
mpc_controller = mpc(sys, Ts, N);

% Input constraints (acceleration limits)
mpc_controller.MV(1).Min = -2;
mpc_controller.MV(1).Max = 2;
mpc_controller.MV(2).Min = -2;
mpc_controller.MV(2).Max = 2;

% Output constraints (velocity limits)
mpc_controller.OV(3).Min = -15;   % vx
mpc_controller.OV(3).Max = 15;
mpc_controller.OV(4).Min = -15;   % vy
mpc_controller.OV(4).Max = 15;

% Weights: prioritize trajectory tracking and fuel efficiency
mpc_controller.Weights.ManipulatedVariables = [0.1 0.1];
mpc_controller.Weights.ManipulatedVariablesRate = [0.1 0.1];
mpc_controller.Weights.OutputVariables = [1 1 0.1 0.1];

% Initial condition
x = [0; 0; 0; 0];

% Reference trajectory: straight diagonal line
xref = [linspace(0, 100, T_sim/Ts + 1);
        linspace(0, 100, T_sim/Ts + 1);
        ones(1, T_sim/Ts + 1)*5;
        ones(1, T_sim/Ts + 1)*5];

% Storage
X = zeros(nx, T_sim/Ts + 1);
X(:,1) = x;
U = zeros(nu, T_sim/Ts);

% Simulation loop
for k = 1:T_sim/Ts
    % Apply external disturbance
    disturbance = [0; 0; 0.1*sin(0.1*k); 0.1*cos(0.1*k)];
    x = x + disturbance;

    % Set reference for current time
    r = xref(:,k+1)';
    
    % Compute control action
    u = mpcmove(mpc_controller, x, r);

    % Update state
    x = A*x + B*u';

    % Log
    X(:,k+1) = x;
    U(:,k) = u;
end

% Plot results
figure;
subplot(2,2,1);
plot(X(1,:), X(2,:), 'b-', 'LineWidth', 2); hold on;
plot(xref(1,:), xref(2,:), 'r--'); grid on;
xlabel('X [m]'); ylabel('Y [m]');
legend('Trajectory', 'Reference');
title('Aircraft Position');

subplot(2,2,2);
plot(0:Ts:T_sim, X(3,:), 'b-', 0:Ts:T_sim, xref(3,:), 'r--'); grid on;
xlabel('Time [s]'); ylabel('Vx [m/s]');
legend('Vx', 'Reference Vx');

subplot(2,2,3);
plot(0:Ts:T_sim, X(4,:), 'b-', 0:Ts:T_sim, xref(4,:), 'r--'); grid on;
xlabel('Time [s]'); ylabel('Vy [m/s]');
legend('Vy', 'Reference Vy');

subplot(2,2,4);
stairs(0:Ts:T_sim-Ts, U'); grid on;
xlabel('Time [s]'); ylabel('Acceleration [m/s^2]');
legend('Ax', 'Ay');
title('Control Inputs');
