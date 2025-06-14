% Define distillation column dynamics
A = [-0.028, 0.018; 0.012, -0.032];
B = [0.02; 0.015];
C = eye(2);
D = zeros(2,1);

% Create state-space model
distColumn = ss(A, B, C, D);

% Discretize model for MPC implementation
Ts = 1; % sampling time (minute)
distColumnD = c2d(distColumn, Ts);

% MPC parameters
predictionHorizon = 10;
controlHorizon = 2;

% Define MPC object
mpcObj = mpc(distColumnD, Ts, predictionHorizon, controlHorizon);

% Define constraints
mpcObj.MV.Min = 0;           % Minimum manipulated variable
mpcObj.MV.Max = 100;         % Maximum manipulated variable
mpcObj.OV.Min = [50; 70];    % Minimum output temperatures
mpcObj.OV.Max = [100; 120];  % Maximum output temperatures

% Weights for MPC tuning
mpcObj.Weights.MV = 0.1;
mpcObj.Weights.MVRate = 0.1;
mpcObj.Weights.OV = [1, 1];

% Simulation parameters
simTime = 50;
refSignal = repmat([80; 100], 1, simTime); % Desired temperature setpoints

% Initial conditions
x0 = [70; 90];

% Initialize arrays
y = zeros(2, simTime);
u = zeros(1, simTime);
x = x0;

% MPC simulation loop
for k = 1:simTime
    % Compute optimal control action
    u(k) = mpcmove(mpcObj, x, y(:,max(k-1,1)), refSignal(:,k));

    % Update states and outputs
    x = distColumnD.A * x + distColumnD.B * u(k);
    y(:,k) = distColumnD.C * x;
end
