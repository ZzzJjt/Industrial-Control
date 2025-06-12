% Clear workspace and command window
clear;
clc;

% Step 1: Define Aircraft Dynamic Model
% Simplified linear model for altitude and velocity control
% States: [Altitude, Velocity]
% Inputs: [Thrust Command]
% Outputs: [Altitude]

% System parameters
A = [1, 0.1; 0, 0.95]; % State matrix
B = [0.01; 0.02]; % Input matrix
C = [1, 0]; % Output matrix
D = [0]; % Direct transmission matrix

% Create state-space model
sys = ss(A, B, C, D);

% Step 2: Define MPC Controller
% Sampling time
Ts = 1; % Sampling time [seconds]
Np = 10; % Prediction horizon
Nm = 3; % Control horizon

% Create MPC controller
mpcobj = mpc(sys, Ts, Np, Nm);
mpcobj.Weights.OutputVariables = 1; % Weight on output tracking
mpcobj.Weights.ManipulatedVariablesRate = 0.1; % Weight on rate of change of inputs

% Set constraints
umin = -1; % Minimum thrust command
umax = 1; % Maximum thrust command
xmin = [0; 0]; % Minimum state [Altitude, Velocity]
xmax = [10000; 500]; % Maximum state [Altitude, Velocity]

% Apply constraints
mpcobj.MV.Min = umin; % Minimum thrust command
mpcobj.MV.Max = umax; % Maximum thrust command
mpcobj.OV(1).Min = xmin(1); % Minimum altitude
mpcobj.OV(1).Max = xmax(1); % Maximum altitude

% Set setpoints
T_setpoint = 5000; % Desired altitude [feet]
V_setpoint = 300; % Desired velocity [ft/s]
setpoint(mpcobj, 'MV', 0); % Thrust command setpoint (initially zero)
setpoint(mpcobj, 'OV', [T_setpoint; V_setpoint]); % Altitude and velocity setpoints

% Step 3: Simulation of Trajectory Scenarios with Disturbances
% Initial conditions
x0 = [0; 0]; % Initial state [Altitude, Velocity]
u0 = 0; % Initial input [Thrust Command]

% Time settings
t_sim = 60; % Simulation time [seconds]
N = t_sim / Ts; % Number of simulation steps

% Disturbances (wind gusts)
disturbance = sin((0:N-1)*Ts/10) * 5; % Wind gusts affecting velocity

% Initialize variables
u = zeros(N, 1); % Inputs [Thrust Command]
y = zeros(N, 1); % Outputs [Altitude]
v = zeros(N, 1); % Velocities
x = x0 * ones(length(x0), N); % States [Altitude, Velocity]

% Simulate the system with MPC control
for k = 1:N-1
    % Get current state
    x(:, k) = x0;
    
    % Compute MPC control action
    r = struct('Ref', [T_setpoint; V_setpoint]);
    mv = mpcmove(mpcobj, x0, r);
    
    % Apply control action
    u(k) = mv';
    
    % Update state using state-space equations with disturbance
    x0 = A * x0 + B * mv + [0; disturbance(k)];
    
    % Measure output
    y(k) = C * x0 + D * mv;
    v(k) = x0(2);
end

% Log final state and inputs
x(:, end) = x0;
u(end) = u(end-1);
y(end) = C * x0 + D * u(end);
v(end) = x0(2);

% Print simulation results
disp('Simulation Results:');
disp(['Time\tAltitude\tVelocity\tThrust Command\tWind Gust']);
for k = 1:N
    fprintf('%d\t%.2f\t\t%.2f\t\t%.2f\t\t%.2f\n', k*Ts, y(k), v(k), u(k), disturbance(k));
end
