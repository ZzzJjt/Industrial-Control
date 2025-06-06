FUNCTION_BLOCK FB_KalmanFilter_PositionVelocity
VAR_INPUT
    MeasuredPosition : REAL;     // Sensor measurement of position
    Q                : REAL;     // Process noise covariance
    R                : REAL;     // Measurement noise covariance
    DeltaT           : REAL;     // Time step
    Reset            : BOOL;     // Reset flag
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL;    // Filtered position estimate
    EstimatedVelocity : REAL;    // Filtered velocity estimate
    KalmanGain        : REAL;    // Optional output for debugging/monitoring
END_VAR

VAR
    x_est_pos : REAL := 0.0;     // Internal position estimate
    x_est_vel : REAL := 0.0;     // Internal velocity estimate
    x_est_prev: REAL := 0.0;     // Previous position estimate
    p         : REAL := 1.0;     // Estimation error covariance
    k         : REAL := 0.0;     // Kalman gain
END_VAR

// Reset logic
IF Reset THEN
    x_est_pos := 0.0;
    x_est_vel := 0.0;
    p := 1.0;
    EstimatedPosition := 0.0;
    EstimatedVelocity := 0.0;
    KalmanGain := 0.0;
ELSE
    // Prediction Step
    x_est_prev := x_est_pos;
    x_est_pos := x_est_pos + DeltaT * x_est_vel;
    p := p + Q;

    // Update Step
    k := p / (p + R);
    x_est_pos := x_est_pos + k * (MeasuredPosition - x_est_pos);
    x_est_vel := (x_est_pos - x_est_prev) / DeltaT;
    p := (1.0 - k) * p;

    // Output
    EstimatedPosition := x_est_pos;
    EstimatedVelocity := x_est_vel;
    KalmanGain := k;
END_IF;
