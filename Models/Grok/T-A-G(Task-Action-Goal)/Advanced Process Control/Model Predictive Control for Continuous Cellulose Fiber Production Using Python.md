function [x_dot, y] = aircraft_model(t, x, u, wind)
    % Aircraft 6-DOF model: states [x, y, z, vx, vy, vz, theta, psi]
    % Inputs: u = [T, delta_e, delta_r] (thrust, elevator, rudder)
    % Wind: [wx, wy, wz] (m/s)
    
    % Parameters
    m = 10000; % Mass (kg)
    g = 9.81; % Gravity (m/s^2)
    rho = 1.225; % Air density (kg/m^3)
    S = 30; % Wing area (m^2)
    CL0 = 0.3; % Lift coefficient base
    CL_delta = 0.5; % Lift coefficient per elevator angle
    CD0 = 0.02; % Drag coefficient base
    k = 0.1; % Induced drag factor
    Iyy = 50000; % Pitch moment of inertia (kg·m^2)
    Izz = 60000; % Yaw moment of inertia (kg·m^2)
    
    % States
    x_pos = x(1); y_pos = x(2); z_pos = x(3); % Position (m)
    vx = x(4); vy = x(5); vz = x(6); % Velocity (m/s)
    theta = x(7); % Pitch angle (rad)
    psi = x(8); % Yaw angle (rad)
    
    % Control inputs
    T = u(1); % Thrust (N)
    delta_e = u(2); % Elevator angle (rad)
    delta_r = u(3); % Rudder angle (rad)
    
    % Aerodynamic forces
    V = sqrt(vx^2 + vy^2 + vz^2); % Airspeed
    q = 0.5 * rho * V^2; % Dynamic pressure
    CL = CL0 + CL_delta * delta_e; % Lift coefficient
    CD = CD0 + k * CL^2; % Drag coefficient
    L = q * S * CL; % Lift force
    D = q * S * CD; % Drag force
    
    % Forces in body frame
    ax = (T * cos(theta) - D - m * g * sin(theta) + wind(1)) / m;
    ay = (T * sin(psi) * sin(theta) + L * cos(psi) + wind(2)) / m;
    az = (-T * cos(psi) * sin(theta) + L * sin(psi) - m * g * cos(theta) + wind(3)) / m;
    
    % Angular accelerations
    dtheta_dt = q * S * delta_e / Iyy; % Pitch rate
    dpsi_dt = q * S * delta_r / Izz; % Yaw rate
    
    % State derivatives
    x_dot = [vx; vy; vz; ax; ay; az; dtheta_dt; dpsi_dt];
    
    % Outputs (position for tracking)
    y = [x_pos; y_pos; z_pos];
end
