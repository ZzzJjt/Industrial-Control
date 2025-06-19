FUNCTION_BLOCK FB_KalmanFilter
VAR_INPUT
    MeasuredPosition : REAL;    // Sensor input
    DeltaT           : REAL;    // Time step in seconds
    Q                : REAL;    // Process noise covariance
    R                : REAL;    // Measurement noise covariance
    Reset            : BOOL;    // Reset the filter
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL;   // Filtered position
    EstimatedVelocity : REAL;   // Filtered velocity
    KalmanGain        : REAL;   // Current Kalman gain (scalar, for position)
END_VAR

VAR
    // State estimates
    X_pos : REAL := 0.0;
    X_vel : REAL := 0.0;

    // Covariance matrix (2x2, only upper triangle stored)
    P_00 : REAL := 1.0; // pos-pos
    P_01 : REAL := 0.0; // pos-vel
    P_11 : REAL := 1.0; // vel-vel

    // Kalman gain components
    K_0  : REAL := 0.0; // For position
    K_1  : REAL := 0.0; // For velocity

    // Intermediate variables
    S    : REAL := 0.0; // Innovation covariance
    Y    : REAL := 0.0; // Measurement residual
END_VAR

// Initialization
IF Reset THEN
    X_pos := 0.0;
    X_vel := 0.0;
    P_00 := 1.0;
    P_01 := 0.0;
    P_11 := 1.0;
END_IF

// --- PREDICTION STEP ---
X_pos := X_pos + DeltaT * X_vel;
// State transition model: [1 Δt; 0 1]
P_00 := P_00 + DeltaT * (2.0 * P_01 + DeltaT * P_11) + Q;
P_01 := P_01 + DeltaT * P_11;
P_11 := P_11 + Q;

// --- UPDATE STEP ---
S := P_00 + R;
IF S <> 0.0 THEN
    KalmanGain := P_00 / S;
    K_0 := KalmanGain;
    K_1 := P_01 / S;
ELSE
    K_0 := 0.0;
    K_1 := 0.0;
END_IF

Y := MeasuredPosition - X_pos;
X_pos := X_pos + K_0 * Y;
X_vel := X_vel + K_1 * Y;

// Update covariance matrix
P_00 := (1.0 - K_0) * P_00;
P_01 := (1.0 - K_0) * P_01;
P_11 := P_11 - K_1 * P_01;

// Output results
EstimatedPosition := X_pos;
EstimatedVelocity := X_vel;
