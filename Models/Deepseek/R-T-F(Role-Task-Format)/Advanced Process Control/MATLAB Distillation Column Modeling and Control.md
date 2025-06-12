% Clear workspace and command window
clear;
clc;

% Step 1: Mathematical Modeling
% Define the state-space model for the distillation column
% States: [Temperature, Reflux Ratio]
% Inputs: [Feed Rate, Feed Temperature]
% Outputs: [Temperature]

% System parameters
A = [0.95, 0.02; 0.01, 0.98]; % State matrix
B = [0.01, 0.005; 0.005, 0.01]; % Input matrix
C = [1, 0]; % Output matrix
D = [0, 0]; % Direct transmission matrix

% Create state-space model
sys = ss(A, B, C, D);

% Step 2: Define System Parameters and Constraints
% Initial conditions
x0 = [70; 1.2]; % Initial temperature and reflux ratio

% Time settings
Ts = 1; % Sampling time [seconds]
t_sim = 60; % Simulation time [seconds]
N = t_sim / Ts; % Number of simulation steps

% Setpoints
T_setpoint = 80; % Desired temperature [°C]
R_setpoint = 1.0; % Desired reflux ratio [-]

% Constraints
umin = [0; 0]; % Minimum input [Feed Rate, Feed Temperature]
umax = [10; 10]; % Maximum input [Feed Rate, Feed Temperature]
xmin = [50; 0.5]; % Minimum state [Temperature, Reflux Ratio]
xmax = [90; 2.0]; % Maximum state [Temperature, Reflux Ratio]

% Step 3: Implement MPC Logic
% Define MPC controller
mpcobj = mpc(sys, Ts);
mpcobj.Weights.OutputVariables = 1; % Weight on output tracking
mpcobj.Weights.ManipulatedVariablesRate = 0.1; % Weight on rate of change of inputs

% Set constraints
mpcobj.MV(1).Min = umin(1); % Minimum feed rate
mpcobj.MV(1).Max = umax(1); % Maximum feed rate
mpcobj.MV(2).Min = umin(2); % Minimum feed temperature
mpcobj.MV(2).Max = umax(2); % Maximum feed temperature
mpcobj.OV(1).Min = xmin(1); % Minimum temperature
mpcobj.OV(1).Max = xmax(1); % Maximum temperature

% Set setpoints
setpoint(mpcobj, 'MV', R_setpoint); % Reflux ratio setpoint
setpoint(mpcobj, 'OV', T_setpoint); % Temperature setpoint

% Step 4: Numerical Simulation
% Initialize variables
u = zeros(2, N); % Inputs [Feed Rate, Feed Temperature]
y = zeros(1, N); % Output [Temperature]
x = x0 * ones(length(x0), N); % States [Temperature, Reflux Ratio]

% Simulate the system with MPC control
for k = 1:N-1
    % Get current state
    x(:, k) = x0;
    
    % Compute MPC control action
    r = struct('Ref', [T_setpoint; R_setpoint]);
    mv = mpcmove(mpcobj, x0, r);
    
    % Apply control action
    u(:, k) = mv';
    
    % Update state using state-space equations
    x0 = A * x0 + B * mv;
    
    % Measure output
    y(k) = C * x0 + D * mv;
end

% Log final state and inputs
x(:, end) = x0;
u(:, end) = u(:, end-1);

% Print simulation results
disp('Simulation Results:');
disp(['Time\tTemperature\tReflux Ratio\tFeed Rate\tFeed Temperature']);
for k = 1:N
    fprintf('%d\t%.2f\t\t%.2f\t\t%.2f\t\t%.2f\n', k*Ts, y(k), x(2,k), u(1,k), u(2,k));
end
