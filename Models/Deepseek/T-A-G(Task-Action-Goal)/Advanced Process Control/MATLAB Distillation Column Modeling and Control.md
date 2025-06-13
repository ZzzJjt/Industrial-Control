% Step 1: Develop the Mathematical Model
%
% Simplified mass and energy balance equations for a binary distillation column:
%
% Mass Balance:
%   dh/dt = F - D - B
%
% Energy Balance:
%   dH/dt = F*h_F + L*(h_L - h_D) - Q
%
% where:
%   h = molar enthalpy
%   H = total enthalpy
%   F = feed flow rate
%   D = distillate flow rate
%   B = bottoms flow rate
%   h_F = enthalpy of feed
%   h_L = enthalpy of liquid reflux
%   h_D = enthalpy of distillate
%   Q = heat input

% Parameters
F = 100;          % Feed flow rate (kmol/h)
x_F = 0.5;        % Mole fraction of component A in feed
T_F = 350;        % Temperature of feed (K)
D_over_F = 0.6;   % Distillate ratio
B_over_F = 0.4;   % Bottoms ratio
L_over_F = 0.8;   % Reflux ratio
Cp_A = 200;       % Heat capacity of component A (J/kmol*K)
Cp_B = 150;       % Heat capacity of component B (J/kmol*K)
Hv_A = 20000;     % Enthalpy of vaporization of component A (J/kmol)
Hv_B = 15000;     % Enthalpy of vaporization of component B (J/kmol)

% Constants
R = 8.314;        % Gas constant (J/mol*K)

% Initial conditions
h0 = 20000;       % Initial molar enthalpy (J/kmol)
H0 = 2000000;     % Initial total enthalpy (J)

% Time span for simulation
tspan = [0 10];    % Simulation time from 0 to 10 hours
dt = 0.1;         % Time step (hours)

% Step 2: Implement the Model in MATLAB
function dydt = distillation_column(t, y, u)
    % Unpack states
    h = y(1);
    H = y(2);
    
    % Unpack control variable
    Q = u;
    
    % Calculate mole fractions based on enthalpy
    x_A = (h - Hv_B * x_F) / (Hv_A * x_F + Hv_B * (1 - x_F));
    x_B = 1 - x_A;
    
    % Calculate enthalpies
    h_F = x_F * Cp_A * T_F + (1 - x_F) * Cp_B * T_F;
    h_D = Hv_A * x_A + Cp_A * T_F + Hv_B * x_B + Cp_B * T_F;
    h_L = Hv_A * x_A + Cp_A * T_F + Hv_B * x_B + Cp_B * T_F;
    
    % Mass balance
    dhdt = F - D_over_F*F - B_over_F*F;
    
    % Energy balance
    dHdt = F*h_F + L_over_F*F*(h_L - h_D) - Q;
    
    % Pack derivatives
    dydt = [dhdt; dHdt];
end

% Step 3: Design the MPC Controller
% Define plant model
A = eye(2); % State matrix
B = [0; 1/(F*Cp_A)]; % Input matrix
C = eye(2); % Output matrix
D = zeros(2,1); % Direct transmission matrix
Ts = dt; % Sample time

plant = ss(A, B, C, D, Ts);

% Define MPC controller
mpcobj = mpc(plant, Ts);
mpcobj.Weights.OutputVariables = 1;
mpcobj.Weights.ManipulatedVariablesRate = 0.1;

% Add constraints
mpcobj.MV.Min = 0; % Minimum heat input
mpcobj.MV.Max = 100000; % Maximum heat input

% Step 4: Simulate the System
[y_mpc, t] = sim(mpcobj, tspan, [h0; H0], [], F);

% Extract temperature profiles
x_A_profile = (y_mpc.Data(:,1) - Hv_B * x_F) ./ (Hv_A * x_F + Hv_B * (1 - x_F));
T_profile = x_A_profile .* Cp_A * T_F + (1 - x_A_profile) .* Cp_B * T_F;

% Display results
disp('Simulation Results:');
disp(['Final Temperature Profile: ', num2str(T_profile(end))]);
disp(['Final Molar Enthalpy Profile: ', num2str(y_mpc.Data(end,1))]);
disp(['Final Total Enthalpy Profile: ', num2str(y_mpc.Data(end,2))]);



