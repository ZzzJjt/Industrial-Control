FUNCTION_BLOCK FB_KalmanFilter
VAR_INPUT
    MeasuredPosition : REAL; // Measured position from sensors
    Q : REAL; // Process noise covariance
    R : REAL; // Measurement noise covariance
    DeltaT : REAL; // Sample time
    Reset : BOOL; // Flag to reset the filter
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL; // Estimated position
    EstimatedVelocity : REAL; // Estimated velocity
    KalmanGain : REAL; // Optional Kalman gain output
END_VAR

VAR
    x_est_pos : REAL := 0.0; // Estimated position state
    x_est_vel : REAL := 0.0; // Estimated velocity state
    x_est_prev_pos : REAL := 0.0; // Previous estimated position state
    x_est_prev_vel : REAL := 0.0; // Previous estimated velocity state
    p : REAL := 1.0; // Estimate error covariance
    k : REAL := 0.0; // Kalman gain
    A : REAL := 1.0; // State transition matrix element (constant)
    B : REAL := 0.0; // Control input matrix element (not used here)
    H : REAL := 1.0; // Measurement matrix element (constant)
    P_pred : REAL := 1.0; // Predicted estimate error covariance
END_VAR

// Method to initialize the Kalman filter states
METHOD Initialize(this : REFERENCE TO FB_KalmanFilter)
BEGIN
    this.x_est_pos := 0.0;
    this.x_est_vel := 0.0;
    this.p := 1.0;
END_METHOD

// Main method to execute the Kalman filter logic
METHOD Execute(this : REFERENCE TO FB_KalmanFilter) : BOOL
BEGIN
    // Validate inputs
    IF this.DeltaT <= 0 OR this.Q < 0 OR this.R < 0 THEN
        RETURN FALSE;
    END_IF;

    // Reset the filter if the Reset flag is set
    IF this.Reset THEN
        this.Initialize();
        RETURN TRUE;
    END_IF;

    // Prediction step
    this.x_est_prev_pos := this.x_est_pos;
    this.x_est_prev_vel := this.x_est_vel;
    this.x_est_pos := this.x_est_prev_pos + this.x_est_prev_vel * this.DeltaT;
    this.x_est_vel := this.x_est_prev_vel;
    this.P_pred := this.p + this.Q;

    // Update (Correction) step
    this.k := this.P_pred / (this.P_pred + this.R);
    this.x_est_pos := this.x_est_pos + this.k * (this.MeasuredPosition - this.H * this.x_est_pos);
    this.x_est_vel := this.x_est_vel;
    this.p := (1.0 - this.k * this.H) * this.P_pred;

    // Output the results
    this.EstimatedPosition := this.x_est_pos;
    this.EstimatedVelocity := this.x_est_vel;
    this.KalmanGain := this.k;

    RETURN TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_KalmanFilter) : BOOL
BEGIN
    RETURN this.Execute();
END_METHOD

PROGRAM MainProgram
VAR
    KalmanFilterFB : FB_KalmanFilter;
    MeasuredPosition : REAL := 10.5; // Example measured position
    Q : REAL := 0.1; // Process noise covariance
    R : REAL := 1.0; // Measurement noise covariance
    DeltaT : REAL := 0.1; // Sample time
    Reset : BOOL := FALSE; // Reset flag
    EstimatedPosition : REAL;
    EstimatedVelocity : REAL;
    KalmanGain : REAL;
END_VAR

// Set the inputs for the Kalman filter
KalmanFilterFB.MeasuredPosition := MeasuredPosition;
KalmanFilterFB.Q := Q;
KalmanFilterFB.R := R;
KalmanFilterFB.DeltaT := DeltaT;
KalmanFilterFB.Reset := Reset;

// Call the Kalman filter function block
KalmanFilterFB();

// Get the estimated position, velocity, and Kalman gain
EstimatedPosition := KalmanFilterFB.EstimatedPosition;
EstimatedVelocity := KalmanFilterFB.EstimatedVelocity;
KalmanGain := KalmanFilterFB.KalmanGain;

// Output the results
// For demonstration purposes, assume there's a way to display or log the results
// Display(EstimatedPosition); // Hypothetical display function
// Display(EstimatedVelocity); // Hypothetical display function
// Display(KalmanGain); // Hypothetical display function
