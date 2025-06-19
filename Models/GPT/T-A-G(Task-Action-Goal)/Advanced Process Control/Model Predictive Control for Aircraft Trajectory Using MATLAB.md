% Aircraft linearized state-space model (simplified longitudinal dynamics)
A = [0 1 0 0;
     0 -0.02 0.1 0;
     0 0 0 1;
     0 -0.1 -0.5 -0.02];
B = [0; 0.1; 0; 0.5];
C = eye(4);
D = zeros(4,1);

% Discrete-time conversion
Ts = 0.1; % sampling time (seconds)
sys_d = c2d(ss(A, B, C, D), Ts);

% MPC setup
predictionHorizon = 20;
controlHorizon = 5;
mpcObj = mpc(sys_d, Ts, predictionHorizon, controlHorizon);

% Constraints on control input (e.g., elevator deflection)
mpcObj.MV.Min = -0.3;
mpcObj.MV.Max =  0.3;

% Output constraints (e.g., altitude and pitch angle limits)
mpcObj.OV(1).Min = -100;  % altitude deviation [m]
mpcObj.OV(1).Max =  100;
mpcObj.OV(3).Min = -10;   % pitch angle [deg]
mpcObj.OV(3).Max =  10;

% Weights for optimization
mpcObj.Weights.MV = 0.01;
mpcObj.Weights.MVRate = 0.1;
mpcObj.Weights.OV = [2 0 5 0];  % emphasize altitude and pitch tracking

% Simulation parameters
simTime = 100;
x = zeros(4,1);
y = zeros(4, simTime);
u = zeros(1, simTime);

% Reference trajectory (altitude step + wind disturbance simulation)
r = zeros(4, simTime);
r(1,:) = 50;  % altitude reference
wind = 0.05 * randn(1, simTime);  % simulate wind disturbance (random noise)

% Run closed-loop MPC simulation
for k = 1:simTime
    % Add wind disturbance to pitch rate
    x(4) = x(4) + wind(k);
    
    % Compute MPC control action
    u(k) = mpcmove(mpcObj, x, y(:,max(k-1,1)), r(:,k));

    % Apply control and update state
    x = sys_d.A * x + sys_d.B * u(k);
    y(:,k) = sys_d.C * x;
end
