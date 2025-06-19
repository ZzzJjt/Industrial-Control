To address the complex task of optimizing an aircraft's trajectory under various constraints and disturbances, we will develop a MATLAB-based Model Predictive Control (MPC) algorithm. This approach allows us to incorporate system dynamics, fuel efficiency goals, external disturbances, and safety/performance constraints into the MPC formulation for dynamic optimization of the aircraft's flight path.

### Step 1: Define the Aircraft Model

First, we need to define a simplified model of the aircraft dynamics. For this example, let’s consider a linearized model around a specific operating point. In practice, you would use a more detailed nonlinear model.

```matlab
% Define the state-space matrices for a simplified aircraft model
A = [0 1; 0 -0.1]; % State matrix
B = [0; 1];        % Input matrix
C = [1 0];         % Output matrix
D = 0;             % Direct transmission matrix

% Create a continuous-time state-space model
sys = ss(A, B, C, D);

% Convert to discrete time with a sample time suitable for your application
Ts = 0.1; % Sample time (seconds)
sysd = c2d(sys, Ts);
```

### Step 2: Design the MPC Controller

Next, design an MPC controller using MATLAB’s Model Predictive Control Toolbox. Set up the prediction and control horizons, specify weights for manipulated variables and output variables, and include constraints relevant to the aircraft operation.

```matlab
% Setup MPC parameters
p = 10; % Prediction horizon
m = 3;  % Control horizon

% Create MPC object
mpcobj = mpc(sysd, Ts, p, m);

% Set weights for manipulated variables and output variables
mpcobj.Weights.ManipulatedVariablesRate = 0.1;
mpcobj.Weights.OutputVariables = [1];

% Constraint settings
mpcobj.MV.Min = -10; % Minimum input value (e.g., throttle or elevator angle)
mpcobj.MV.Max = 10;  % Maximum input value
```

### Step 3: Simulate Disturbances and Optimize Trajectory

Simulate the closed-loop system response under MPC control, considering external disturbances such as wind gusts. We simulate a sudden disturbance in the form of a wind gust and show how the MPC adjusts the trajectory in real-time.

```matlab
% Simulation settings
Tstop = 50; % Total simulation time
r = ones(Tstop, 1)*[1]; % Reference signal for desired altitude or speed

% Initial states
x0 = [0; 0]; % Initial states

% Simulate a disturbance (wind gust) at t=10 seconds
disturbance = zeros(Tstop, 1);
disturbance(10/Ts:end) = 1; % Example disturbance starting at t=10

% Modify reference signal to include disturbance effect
r_disturbed = r + disturbance;

% Run closed-loop simulation without plotting or visualization
[y, t, u] = sim(mpcobj, Tstop, [], r_disturbed, x0);

% Since plotting or visualization is not required, we omit those parts.
```

### Explanation

- **Step 1:** Defines a simple linear model of the aircraft dynamics. In reality, you'd have a more sophisticated nonlinear model.
- **Step 2:** Designs an MPC controller that considers both the predictive capability and the operational constraints of the aircraft.
- **Step 3:** Demonstrates how the MPC responds to disturbances like wind gusts by adjusting the control inputs in real-time to maintain the desired trajectory while ensuring safety and efficiency.

This code framework provides a basic setup for implementing an MPC algorithm to optimize an aircraft's trajectory dynamically. By incorporating detailed models of the aircraft and its environment, along with precise constraints and objectives, this approach can significantly enhance flight performance and safety under varying conditions. 

Note that for actual applications, it's essential to refine the model based on real-world data and test the system extensively under different scenarios to ensure robustness and reliability.
