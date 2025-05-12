function aircraft_mpc_trajectory()
    % Aircraft Trajectory MPC - Numerical Implementation
    
    %% 1. Aircraft Dynamics Model (Simplified 3DOF)
    % States: [x; y; z; v; gamma; psi; mass]
    % Inputs: [thrust; lift_coeff; bank_angle]
    % Disturbances: [wind_x; wind_y; thermal]
    
    % Continuous-time dynamics
    function dx = aircraft_dynamics(~, x, u, d)
        % Constants
        g = 9.81;   % m/s^2
        rho = 1.225; % kg/m^3
        S = 30;      % wing area m^2
        
        % States
        v = x(4);       % velocity (m/s)
        gamma = x(5);   % flight path angle (rad)
        psi = x(6);      % heading angle (rad)
        m = x(7);        % mass (kg)
        
        % Controls
        T = u(1);        % thrust (N)
        CL = u(2);       % lift coefficient
        phi = u(3);      % bank angle (rad)
        
        % Disturbances
        wind_x = d(1);
        wind_y = d(2);
        thermal = d(3);
        
        % Aerodynamic forces
        L = 0.5*rho*v^2*S*CL;  % lift force
        D = 0.5*rho*v^2*S*(0.02 + 0.05*CL^2); % drag force
        
        % Dynamics equations
        dx = zeros(7,1);
        dx(1) = v*cos(gamma)*cos(psi) + wind_x; % x position
        dx(2) = v*cos(gamma)*sin(psi) + wind_y; % y position
        dx(3) = v*sin(gamma) + thermal;         % altitude
        dx(4) = (T - D)/m - g*sin(gamma);       % velocity
        dx(5) = (L*cos(phi) - m*g*cos(gamma))/(m*v); % flight path angle
        dx(6) = L*sin(phi)/(m*v*cos(gamma));        % heading angle
        dx(7) = -0.1*sqrt(T);                      % mass (fuel burn)
    end

    % Discretize using 4th-order Runge-Kutta
    function x_next = discrete_dynamics(x, u, d, dt)
        k1 = aircraft_dynamics(0, x, u, d);
        k2 = aircraft_dynamics(0, x + dt/2*k1, u, d);
        k3 = aircraft_dynamics(0, x + dt/2*k2, u, d);
        k4 = aircraft_dynamics(0, x + dt*k3, u, d);
        x_next = x + dt/6*(k1 + 2*k2 + 2*k3 + k4);
    end

    %% 2. MPC Setup
    Ts = 5; % Sampling time (seconds)
    N = 10; % Prediction horizon
    M = 3;  % Control horizon
    
    % State and input bounds
    x_min = [-inf; -inf; 500; 80; -0.2; -pi; 5000];
    x_max = [ inf;  inf; 5000; 300; 0.2; pi; 10000];
    u_min = [0; 0.1; -pi/4];
    u_max = [50000; 1.5; pi/4];
    
    % Reference trajectory (waypoints)
    ref_traj = generate_reference_trajectory();
    
    % Initialize
    x0 = [0; 0; 1000; 150; 0; pi/4; 8000]; % Initial state
    x = x0;
    u_prev = [20000; 0.5; 0]; % Initial control
    
    %% 3. Simulation Loop
    for k = 1:120 % 10 minute simulation
        
        % Get current reference
        current_ref = ref_traj(:,k:min(k+N-1,size(ref_traj,2)));
        
        % Get disturbances (wind + thermals)
        d = get_disturbances(k);
        
        % Solve MPC optimization
        u_opt = solve_mpc(x, u_prev, current_ref, d, N, M, Ts);
        
        % Apply first control input
        u = u_opt(:,1);
        x = discrete_dynamics(x, u, d, Ts);
        u_prev = u;
        
        % Display current status
        fprintf('Step %d: Pos=[%.1f, %.1f, %.1f]m, Vel=%.1fm/s\n',...
                k, x(1), x(2), x(3), x(4));
        fprintf('        Controls: Thrust=%.1fN, CL=%.2f, Bank=%.2frad\n',...
                u(1), u(2), u(3));
        fprintf('        Fuel remaining: %.1fkg\n\n', x(7));
    end
    
    %% Nested Functions
    function ref = generate_reference_trajectory()
        % Create a climbing turn scenario
        steps = 120;
        ref = zeros(7, steps);
        
        for t = 1:steps
            angle = pi/4 + 0.01*t;
            alt = 1000 + 10*t;
            ref(:,t) = [500*t*cos(angle);  % x
                        500*t*sin(angle);  % y
                        alt;               % z
                        180;               % v
                        0.05;              % gamma
                        angle;             % psi
                        nan];              % mass (not tracked)
        end
    end

    function d = get_disturbances(k)
        % Simulate changing wind conditions
        wind_x = 5*sin(k/20);
        wind_y = 3*cos(k/15);
        thermal = 0.5*sin(k/10);
        d = [wind_x; wind_y; thermal];
    end

    function u_opt = solve_mpc(x, u_prev, ref, d, N, M, Ts)
        % Formulate and solve MPC optimization
        
        % Create optimization variables
        opti = casadi.Opti();
        X = opti.variable(7,N+1);
        U = opti.variable(3,M);
        
        % Initial state constraint
        opti.subject_to(X(:,1) == x);
        
        % Dynamics constraints
        for k = 1:N
            if k <= M
                u = U(:,k);
            else
                u = U(:,end); % Hold last control
            end
            
            % Apply dynamics
            x_next = discrete_dynamics(X(:,k), u, d, Ts);
            
            % State constraints
            opti.subject_to(X(:,k+1) == x_next);
            opti.subject_to(x_min <= X(:,k+1) <= x_max);
        end
        
        % Input constraints
        opti.subject_to(u_min <= U <= u_max);
        opti.subject_to(-0.5 <= diff(U(1,:)) <= 0.5); % Thrust rate limit
        opti.subject_to(-0.1 <= diff(U(2,:)) <= 0.1); % CL rate limit
        opti.subject_to(-0.2 <= diff(U(3,:)) <= 0.2); % Bank rate limit
        
        % Objective function
        J = 0;
        for k = 1:N
            % Tracking error (position, velocity)
            J = J + 10*(X(1:3,k)-ref(1:3,k))'*(X(1:3,k)-ref(1:3,k));
            J = J + 5*(X(4,k)-ref(4,k))^2;
            
            % Fuel cost (minimize thrust)
            if k <= M
                J = J + 0.01*U(1,k)^2;
            end
            
            % Passenger comfort (smooth flight)
            if k > 1
                J = J + 100*X(5,k)^2 + 50*U(3,k)^2;
            end
        end
        
        % Solve optimization
        opti.minimize(J);
        opti.solver('ipopt');
        sol = opti.solve();
        
        u_opt = sol.value(U);
    end
end
