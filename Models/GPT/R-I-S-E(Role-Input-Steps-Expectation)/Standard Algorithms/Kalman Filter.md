FUNCTION_BLOCK FB_KalmanFilter_PosVel
VAR_INPUT
    MeasuredPosition : REAL;   // Current position measurement
    Q : REAL;                  // Process noise covariance
    R : REAL;                  // Measurement noise covariance
    DeltaT : REAL;            // Sample time (e.g., 0.1 sec)
    Reset : BOOL;              // Reset the filter
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL;  // Filtered position output
    EstimatedVelocity : REAL;  // Filtered velocity output
END_VAR

VAR
    // State variables
    x_est_pos : REAL := 0.0;
    x_est_vel : REAL := 0.0;
    x_est_prev_pos : REAL := 0.0;

    // Error covariance
    p_pos : REAL := 1.0; // Position estimation uncertainty
    p_vel : REAL := 1.0; // Velocity estimation uncertainty

    // Kalman gains
    k_pos : REAL := 0.0;
    k_vel : REAL := 0.0;

    // Predicted state
    x_pred_pos : REAL;
    x_pred_vel : REAL;
    p_pred_pos : REAL;
    p_pred_vel : REAL;

    // Measurement residual
    y : REAL;

    // Auxiliary
    pos_diff : REAL;
END_VAR

IF Reset THEN
    x_est_pos := MeasuredPosition;
    x_est_vel := 0.0;
    x_est_prev_pos := MeasuredPosition;
    p_pos := 1.0;
    p_vel := 1.0;
ELSE
    //--- Prediction ---
    x_pred_pos := x_est_pos + x_est_vel * DeltaT;
    x_pred_vel := x_est_vel;

    p_pred_pos := p_pos + Q;
    p_pred_vel := p_vel + Q;

    //--- Correction ---
    // Innovation (residual)
    y := MeasuredPosition - x_pred_pos;

    // Kalman gains
    k_pos := p_pred_pos / (p_pred_pos + R);
    k_vel := p_pred_vel / (p_pred_vel + R); // Optional use

    // Update estimates
    x_est_pos := x_pred_pos + k_pos * y;

    // Estimate velocity using position difference
    pos_diff := x_est_pos - x_est_prev_pos;
    x_est_vel := pos_diff / DeltaT;
    x_est_prev_pos := x_est_pos;

    // Update error covariances
    p_pos := (1.0 - k_pos) * p_pred_pos;
    p_vel := (1.0 - k_vel) * p_pred_vel;
END_IF

// Output assignments
EstimatedPosition := x_est_pos;
EstimatedVelocity := x_est_vel;
