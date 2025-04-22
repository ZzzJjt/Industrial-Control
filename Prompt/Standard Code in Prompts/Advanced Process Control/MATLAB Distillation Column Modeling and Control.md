
% Define constants and parameters
numTrays = 10; % Number of trays
feedRate = 100; % Feed flow rate (arbitrary units)
refluxRatio = 2; % Reflux ratio
boilUpRate = 2; % Boil-up ratio

% Initialize states (temperature at each tray)
x0 = 300 * ones(numTrays, 1); % Initial temperatures (in Kelvin)

% Define system matrices for a simple first-order model
A = eye(numTrays) * (1 - 1/numTrays) + diag(ones(1, numTrays-1), 1) * (1/numTrays);
B = zeros(numTrays, 1);
B(1) = 1;

% System model
Ts = 0.1; % Sampling time
sys = c2d(ss(A, B, eye(numTrays), zeros(numTrays, 1)), Ts);
% Define MPC parameters
mpcHorizon = 10; % Prediction horizon
controlHorizon = 2; % Control horizon
m = mpc(sys, Ts, mpcHorizon, controlHorizon);

% Set weights on outputs and manipulated variables
m.Weights.OutputVariables = 1;
m.Weights.ManipulatedVariables = 0.1;

% Define constraints
m.MV = struct('Min', 0, 'Max', 200);
m.OV = struct('Min', 300, 'Max', 400);

% Initial condition of the plant
x = x0;

% Reference trajectory
r = 350 * ones(m.Model.N, 1); % Reference temperature
% Simulation loop
N = 100; % Number of simulation steps
y = zeros(N, numTrays);
u = zeros(N, 1);

for t = 0:N-1
    % Update the measured output
    y(t+1,:) = x';
    
    % Compute the MPC action
    u(t+1) = mpcmove(m, x, y(t+1,:), r);
    
    % Update the state
    x = sys.A * x + sys.B * u(t+1);
end
