FUNCTION_BLOCK KalmanFilterFB
VAR_INPUT
    MeasuredPosition : REAL; // Measured position from sensors
    Q : REAL;                // Process noise covariance
    R : REAL;                // Measurement noise covariance
    DeltaT : REAL;           // Sample time
    Reset : BOOL;            // Reset flag to initialize states
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL; // Estimated position
    EstimatedVelocity : REAL; // Estimated velocity
END_VAR

VAR
    x_est_pos : REAL := 0.0;  // Estimated position state
    x_est_vel : REAL := 0.0;  // Estimated velocity state
    p : REAL := 1.0;          // Error covariance
    k : REAL := 0.0;          // Kalman gain
    x_pred_pos : REAL;        // Predicted position
    x_pred_vel : REAL;        // Predicted velocity
    p_pred : REAL;            // Predicted error covariance
END_VAR

METHOD Execute : VOID
BEGIN
    // Reset logic
    IF Reset THEN
        x_est_pos := 0.0;
        x_est_vel := 0.0;
        p := 1.0;
        RETURN;
    END_IF;

    // Prediction step
    x_pred_pos := x_est_pos + x_est_vel * DeltaT;
    x_pred_vel := x_est_vel;
    p_pred := p + Q;

    // Correction step
    k := p_pred / (p_pred + R);
    x_est_pos := x_pred_pos + k * (MeasuredPosition - x_pred_pos);
    x_est_vel := x_pred_vel + k * ((MeasuredPosition - x_pred_pos) / DeltaT);
    p := (1.0 - k) * p_pred;

    // Update outputs
    EstimatedPosition := x_est_pos;
    EstimatedVelocity := x_est_vel;
END_METHOD

END_FUNCTION_BLOCK
