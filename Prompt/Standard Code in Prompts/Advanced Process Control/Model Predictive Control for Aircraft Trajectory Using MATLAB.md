```
% Constants and parameters
g = 9.81; % Gravity (m/s^2)
mass = 70000; % Mass of the aircraft (kg)
rho = 1.225; % Air density (kg/m^3)
S = 150; % Wing surface area (m^2)
Cd0 = 0.02; % Zero-lift drag coefficient
ClAlpha = 0.1; % Lift curve slope (N/(rad*m^2))

% Initial conditions
h0 = 10000; % Initial altitude (m)
v0 = 250; % Initial airspeed (m/s)
x0 = [h0; v0]; % State vector [altitude, airspeed]

% Time step
Ts = 1; % Sample time (seconds)

% System matrices for linearized dynamics
A = [0 1; ...
     0 -(rho*S*ClAlpha*v0)/(mass*g)];
B = [0; ...
     (rho*S*v0)/(mass*g)];

% Discretize the system
sys = c2d(ss(A, B, eye(2), zeros(2,1)), Ts);
% MPC parameters
predictionHorizon = 20; % Prediction horizon (steps)
controlHorizon = 3; % Control horizon (steps)

% Create MPC controller
mpcController = mpc(sys, Ts, predictionHorizon, controlHorizon);

% Set weights on outputs and manipulated variables
mpcController.Weights.OutputVariables = [1 1]; % Weight on altitude and airspeed
mpcController.Weights.ManipulatedVariables = [0.1]; % Weight on thrust change

% Define constraints
mpcController.MV = struct('Min', 0, 'Max', 1); % Thrust limits (0 to 1)
mpcController.OV = struct('Min', [8000; 200], 'Max', [12000; 300]); % Altitude and speed limits

% Initial condition
x = x0;

% Reference trajectory
refTraj = [12000; 280]; % Desired altitude and airspeed
% Simulation parameters
Nsim = 100; % Number of simulation steps
Y = zeros(Nsim, 2); % Store outputs
U = zeros(Nsim, 1); % Store inputs

for t = 0:Nsim-1
    % Update the measured output
    Y(t+1,:) = x';
    
    % Compute the MPC action
    U(t+1) = mpcmove(mpcController, x, Y(t+1,:), refTraj);
    
    % Simulate the system
    x = sys.A * x + sys.B * U(t+1);
end
```
