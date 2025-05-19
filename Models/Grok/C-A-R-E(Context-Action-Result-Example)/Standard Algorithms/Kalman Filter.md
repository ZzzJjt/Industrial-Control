FUNCTION_BLOCK KalmanFilterFB
VAR_INPUT
    Enable: BOOL;               // Enable filter operation
    Reset: BOOL;                // Reset filter to initial state
    MeasuredPosition: REAL;      // Noisy position measurement (m)
    Q: REAL := 0.01;            // Process noise covariance
    R: REAL := 1.0;             // Measurement noise covariance
    DeltaT: REAL := 0.1;        // Time step (s)
END_VAR

VAR_OUTPUT
    EstimatedPosition: REAL;     // Filtered position estimate (m)
    EstimatedVelocity: REAL;     // Filtered velocity estimate (m/s)
    KalmanGain: REAL;           // Kalman gain for diagnostics
    Error: BOOL;                // TRUE if invalid input
END_VAR

VAR
    x_est_pos: REAL := 0.0;     // Estimated position (internal state)
    x_est_vel: REAL := 0.0;     // Estimated velocity (internal state)
    x_est_prev: REAL := 0.0;    // Previous position for velocity calculation
    p: REAL := 1.0;             // Position covariance
    k: REAL := 0.0;             // Kalman gain (internal)
    ValidInput: BOOL;           // Input validation flag
    Initialized: BOOL := FALSE; // Tracks initialization state
END_VAR

// Reset outputs and state when disabled
IF NOT Enable THEN
    EstimatedPosition := 0.0;
    EstimatedVelocity := 0.0;
    KalmanGain := 0.0;
    Error := FALSE;
    x_est_pos := 0.0;
    x_est_vel := 0.0;
    x_est_prev := 0.0;
    p := 1.0;
    k := 0.0;
    Initialized := FALSE;
    RETURN;
END_IF;

// Input validation
ValidInput := TRUE;
IF Q <= 0.0 OR R <= 0.0 OR DeltaT <= 0.0 THEN
    ValidInput := FALSE;
    Error := TRUE;
    EstimatedPosition := 0.0;
    EstimatedVelocity := 0.0;
    KalmanGain := 0.0;
    RETURN;
END_IF;

// Handle reset
IF Reset THEN
    x_est_pos := MeasuredPosition;
    x_est_vel := 0.0;
    x_est_prev := MeasuredPosition;
    p := 1.0;
    k := 0.0;
    Initialized := TRUE;
END_IF;

// Kalman filter logic
IF ValidInput AND (Initialized OR Reset) THEN
    // Prediction step
    x_est_pos := x_est_pos + DeltaT * x_est_vel; // Predict position
    p := p + Q;                                  // Predict covariance

    // Correction step
    k := p / (p + R);                            // Compute Kalman gain
    x_est_pos := x_est_pos + k * (MeasuredPosition - x_est_pos); // Update position
    p := (1.0 - k) * p;                          // Update covariance

    // Estimate velocity based on position change
    x_est_vel := (x_est_pos - x_est_prev) / DeltaT;
    x_est_prev := x_est_pos;

    // Update outputs
    EstimatedPosition := x_est_pos;
    EstimatedVelocity := x_est_vel;
    KalmanGain := k;
ELSE
    // Initialize on first valid measurement
    IF ValidInput AND NOT Initialized THEN
        x_est_pos := MeasuredPosition;
        x_est_prev := MeasuredPosition;
        x_est_vel := 0.0;
        p := 1.0;
        k := 0.0;
        Initialized := TRUE;
        EstimatedPosition := x_est_pos;
        EstimatedVelocity := x_est_vel;
        KalmanGain := k;
    END_IF;
END_IF;
END_FUNCTION_BLOCK
