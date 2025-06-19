% Step 1: Develop the Mathematical Model
%
% Simplified linearized model of an aircraft:
%
% State variables:
%   x = [altitude, velocity, heading]'
%
% Inputs:
%   u = [throttle, elevator]'
%
% Dynamics:
%   dx/dt = A*x + B*u + D*d
%
% where d is the disturbance due to wind

% Parameters
g = 9.81;          % Gravity (m/s^2)
rho = 1.225;       % Air density (kg/m^3)
S = 40;            % Wing area (m^2)
CD0 = 0.02;        % Zero-lift drag coefficient
CL0 = 0.5;         % Lift curve slope
e = 0.8;           % Oswald efficiency factor
W = 10000;         % Weight (N)

% Linearization point
V0 = 50;           % Nominal velocity (m/s)
h0 = 1000;         % Nominal altitude (m)
theta0 = 0;        % Nominal pitch angle (rad)
gamma0 = 0;        % Nominal flight path angle (rad)
phi0 = 0;          % Nominal roll angle (rad)

% Jacobian matrices at the nominal point
A = [
    0, 0, V0*cos(gamma0);
    -(g/V0)*sin(theta0), -g/V0, g*V0*cos(theta0)/W;
    0, 0, 0
];

B = [
    0, 0;
    (2*rho*S*CL0*e)/(W*sqrt(pi)), -(2*rho*S*CD0)/(W*sqrt(pi));
    0, 0
];

D = [
    0;
    -(2*rho*S*CL0*e)/(W*sqrt(pi)); % Wind effect on lift
    0
];

C = eye(3); % Measurement matrix
Ts = 0.1;   % Sample time (s)

plant = ss(A, B, C, D, Ts);

% Step 2: Define Constraints and Objectives
% Define MPC controller
mpcobj = mpc(plant, Ts);
mpcobj.Weights.ManipulatedVariablesRate = 0.1; % Penalize changes in throttle and elevator
mpcobj.Weights.StateOutput = [1, 1, 1];        % Penalize deviations from reference

% Add constraints
mpcobj.MV.Min = [0; -0.1]; % Minimum throttle and elevator deflection
mpcobj.MV.Max = [1; 0.1];  % Maximum throttle and elevator deflection

mpcobj.OV.Target = [h0; V0; phi0]; % Reference targets for altitude, velocity, and heading

% Step 3: Incorporate External Disturbances
% Assume wind gust as a disturbance
disturbance_model = ss(zeros(3,3), zeros(3,1), [1; 0; 0], 0, Ts); % Wind affects only altitude
addUnmeasuredDisturbance(mpcobj, 'unmeasured', disturbance_model, 1);

% Step 4: Implement the MPC Controller
% Define initial conditions
x0 = [h0; V0; phi0]; % Initial state: altitude, velocity, heading
u0 = [0.5; 0];       % Initial input: throttle, elevator

% Simulate with wind gust
wind_gust = @(t) sin(t) * 5; % Example wind gust function

% Step 5: Simulate Realistic Flight Scenarios
[t, y, u] = sim(mpcobj, 100, x0, [], @wind_gust);

% Extract states and inputs
altitude = y.Data(:,1);
velocity = y.Data(:,2);
heading = y.Data(:,3);
throttle = u.Data(:,1);
elevator = u.Data(:,2);

% Display results
disp('Simulation Results:');
disp(['Final Altitude: ', num2str(altitude(end))]);
disp(['Final Velocity: ', num2str(velocity(end))]);
disp(['Final Heading: ', num2str(heading(end))]);

% Print some intermediate results
disp('Intermediate Results:');
for i = 1:10:length(t)
    disp(['Time: ', num2str(t(i)), ...
           ', Altitude: ', num2str(altitude(i)), ...
           ', Velocity: ', num2str(velocity(i)), ...
           ', Heading: ', num2str(heading(i)), ...
           ', Throttle: ', num2str(throttle(i)), ...
           ', Elevator: ', num2str(elevator(i))]);
end



