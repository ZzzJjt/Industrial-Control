function distillation_mpc()
    % Distillation Column MPC Control - Numerical Implementation Only
    
    %% 1. Distillation Column Model Definition
    % State-space representation of distillation column dynamics
    % States: [Tray temperatures (5 trays), Reboiler level, Condenser level]
    % Inputs: [Feed flow rate, Reboiler heat, Condenser cooling]
    % Outputs: [Top temperature, Bottom temperature, Purity]
    
    % Continuous-time model (simplified for illustration)
    A = [-0.8  0.2    0      0      0    0    0;
         0.3  -0.9   0.3     0      0    0    0;
          0    0.4  -1.0    0.4     0    0    0;
          0     0    0.5   -1.2    0.3   0    0;
          0     0     0     0.6   -1.5   0    0;
         0.1   0.1   0.1    0.1    0.1  -0.5  0;
         0     0     0      0      0    0   -0.7];
     
    B = [0.5  0   0;
         0.2  0   0;
         0.1  0   0;
         0    0.8 0;
         0    0   0;
         0    0   0.4;
         0.3  0   0.6];
     
    C = [0 0 0 0 1 0 0;  % Top temp (last tray)
         1 0 0 0 0 0 0;  % Bottom temp (first tray)
         0 0 0 0 0 0 1]; % Purity (condenser level proxy)
     
    D = zeros(3,3);
    
    % Convert to discrete-time with 1-minute sampling
    sys = ss(A,B,C,D);
    Ts = 60; % Sampling time (seconds)
    sysd = c2d(sys,Ts);
    
    %% 2. MPC Controller Setup
    mpcobj = mpc(sysd, Ts);
    
    % Set prediction and control horizons
    mpcobj.PredictionHorizon = 20;
    mpcobj.ControlHorizon = 5;
    
    % Input constraints (umin < u < umax)
    mpcobj.MV(1).Min = 50;   % Feed flow rate (kg/min)
    mpcobj.MV(1).Max = 150;
    mpcobj.MV(2).Min = 0;    % Reboiler heat (kW)
    mpcobj.MV(2).Max = 100;
    mpcobj.MV(3).Min = 0;     % Condenser cooling (kW)
    mpcobj.MV(3).Max = 80;
    
    % Output constraints (ymin < y < ymax)
    mpcobj.OV(1).Min = 70;    % Top temp (°C)
    mpcobj.OV(1).Max = 90;
    mpcobj.OV(2).Min = 95;    % Bottom temp (°C)
    mpcobj.OV(2).Max = 110;
    mpcobj.OV(3).Min = 0.85;  % Purity (fraction)
    
    % Weights (priority tuning)
    mpcobj.Weights.OV = [1 1 2];      % Output weights
    mpcobj.Weights.MV = [0.1 0.1 0.1]; % Input weights
    mpcobj.Weights.MVRate = [0.5 0.5 0.5]; % Input rate weights
    
    %% 3. Simulation Parameters
    T = 120; % Simulation steps (2 hours)
    yref = [80; 100; 0.9]; % Reference setpoints
    
    % Initialize variables
    x = zeros(size(A,1),1); % Initial state
    u = [80; 40; 30];      % Initial inputs
    
    % Storage for results
    Y = zeros(3,T);
    U = zeros(3,T);
    X = zeros(size(A,1),T);
    
    %% 4. Closed-Loop Simulation
    for k = 1:T
        % Introduce feed composition disturbance every 30 minutes
        if mod(k,30) == 0
            x(1) = x(1) + 5*randn(); % Affect bottom temperature
        end
        
        % MPC control calculation
        u = mpcmove(mpcobj,x,yref);
        
        % Apply control and update state
        x = sysd.A*x + sysd.B*u;
        y = sysd.C*x + sysd.D*u;
        
        % Store results
        Y(:,k) = y;
        U(:,k) = u;
        X(:,k) = x;
        
        % Display current step information
        fprintf('Step %d: Top=%.1f°C, Bottom=%.1f°C, Purity=%.3f\n',...
                k, y(1), y(2), y(3));
        fprintf('        Controls: Feed=%.1f, Reboiler=%.1f, Condenser=%.1f\n\n',...
                u(1), u(2), u(3));
    end
    
    %% 5. Performance Metrics Calculation
    % Calculate average tracking errors
    top_temp_error = mean(abs(Y(1,:) - yref(1)));
    bottom_temp_error = mean(abs(Y(2,:) - yref(2)));
    purity_error = mean(abs(Y(3,:) - yref(3)));
    
    % Calculate constraint violations
    top_temp_viol = sum(Y(1,:) < 70 | Y(1,:) > 90);
    bottom_temp_viol = sum(Y(2,:) < 95 | Y(2,:) > 110);
    purity_viol = sum(Y(3,:) < 0.85);
    
    % Display summary
    fprintf('\n=== Performance Summary ===\n');
    fprintf('Average Tracking Errors:\n');
    fprintf('  Top Temp: %.2f°C\n', top_temp_error);
    fprintf('  Bottom Temp: %.2f°C\n', bottom_temp_error);
    fprintf('  Purity: %.4f\n', purity_error);
    
    fprintf('\nConstraint Violations:\n');
    fprintf('  Top Temp: %d steps\n', top_temp_viol);
    fprintf('  Bottom Temp: %d steps\n', bottom_temp_viol);
    fprintf('  Purity: %d steps\n', purity_viol);
    
    fprintf('\nControl Effort:\n');
    fprintf('  Feed Flow: Avg=%.1f, Min=%.1f, Max=%.1f\n', ...
            mean(U(1,:)), min(U(1,:)), max(U(1,:)));
    fprintf('  Reboiler Heat: Avg=%.1f, Min=%.1f, Max=%.1f\n', ...
            mean(U(2,:)), min(U(2,:)), max(U(2,:)));
    fprintf('  Condenser Cooling: Avg=%.1f, Min=%.1f, Max=%.1f\n', ...
            mean(U(3,:)), min(U(3,:)), max(U(3,:)));
end
