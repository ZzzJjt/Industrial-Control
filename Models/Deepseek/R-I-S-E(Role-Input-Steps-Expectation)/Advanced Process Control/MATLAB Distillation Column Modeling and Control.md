% Distillation Column Model Parameters
A = -0.5;      % First-order system time constant inverse
B = 0.8;       % Input gain (feed rate to temperature)
C = [1; 0.3];  % Output matrix: T and inferred composition
D = 0;         % Direct feedthrough

% Create state-space model
sys = ss(A, B, C, D);

% Discretize for digital control (e.g., sample time Ts = 1 min)
Ts = 1;
sysd = c2d(sys, Ts);

% Define MPC controller
mpcobj = mpc(sysd, Ts);  % Use discretized model

% Set prediction and control horizons
mpcobj.PredictionHorizon = 10;
mpcobj.ControlHorizon = 3;

% Define constraints
% Input (Feed Rate): Min = 10, Max = 100
mpcobj.MV.Min = 10;
mpcobj.MV.Max = 100;
mpcobj.MV.RateMin = -10;   % Rate of change constraint
mpcobj.MV.RateMax = 10;

% Output (Temperature): Min = 90°C, Max = 110°C
mpcobj.OV(1).Min = 90;
mpcobj.OV(1).Max = 110;

% Output (Composition estimate): Min = 0.7, Max = 0.95
mpcobj.OV(2).Min = 0.7;
mpcobj.OV(2).Max = 0.95;

% Optional: Adjust weights
mpcobj.Weights.MV = 0.1;        % Penalize control moves
mpcobj.Weights.MVrate = 0.5;
mpcobj.Weights.OV = [1, 1];     % Track both outputs tightly

% Simulation time
Tstop = 60;  % minutes
time = 0:Ts:Tstop;

% Initial condition
x0 = 95;  % Initial temperature

% Reference setpoints
setpoint_T = 100 * ones(size(time));
setpoint_comp = 0.85 * ones(size(time));
ref = [setpoint_T', setpoint_comp'];  % Two outputs

% Simulate closed-loop response
[~, ~, u] = sim(mpcobj, length(time), ref, 'on');

% Display numerical results
fprintf('Time\tFeed Rate\tTemp\tComposition\n');
for k = 1:length(time)
    fprintf('%d\t%.2f\t\t%.2f\t%.2f\n', ...
            time(k), u(k), sysd.C * x0 + sysd.D * u(k), 0.3 * x0 + 0.3 * u(k));
    % Update state
    x0 = sysd.A * x0 + sysd.B * u(k);
end
