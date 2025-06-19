FUNCTION_BLOCK FB_KalmanFilter2D
VAR_INPUT
    MeasuredPosition : REAL;     // Noisy position measurement
    DeltaT           : REAL;     // Time step in seconds
    Q                : REAL;     // Process noise covariance
    R                : REAL;     // Measurement noise covariance
    Reset            : BOOL;     // Resets filter states
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL;    // Filtered position
    EstimatedVelocity : REAL;    // Filtered velocity
END_VAR

VAR
    // States
    X_Pos      : REAL := 0.0;    // Estimated position
    X_Vel      : REAL := 0.0;    // Estimated velocity

    // Covariance terms (error estimate)
    P_00       : REAL := 1.0;    // Variance of position
    P_01       : REAL := 0.0;
    P_10       : REAL := 0.0;
    P_11       : REAL := 1.0;    // Variance of velocity

    // Kalman Gain
    K_0        : REAL := 0.0;
    K_1        : REAL := 0.0;

    // Other intermediate variables
    Y          : REAL := 0.0;    // Measurement residual
    S          : REAL := 0.0;    // Residual covariance
    Temp_00    : REAL;
    Temp_01    : REAL;
    Temp_10    : REAL;
    Temp_11    : REAL;
END_VAR

// Reset logic
IF Reset THEN
    X_Pos := 0.0;
    X_Vel := 0.0;
    P_00 := 1.0;
    P_01 := 0.0;
    P_10 := 0.0;
    P_11 := 1.0;
    EstimatedPosition := 0.0;
    EstimatedVelocity := 0.0;
ELSE
    // === PREDICTION STEP ===
    X_Pos := X_Pos + DeltaT * X_Vel;  // Predict new position
    // Velocity assumed constant → no update

    // Predict covariance
    Temp_00 := P_00 + DeltaT * (P_10 + P_01) + DeltaT * DeltaT * P_11 + Q;
    Temp_01 := P_01 + DeltaT * P_11;
    Temp_10 := P_10 + DeltaT * P_11;
    Temp_11 := P_11 + Q;

    P_00 := Temp_00;
    P_01 := Temp_01;
    P_10 := Temp_10;
    P_11 := Temp_11;

    // === UPDATE STEP ===
    S := P_00 + R;              // Innovation covariance
    Y := MeasuredPosition - X_Pos;  // Innovation (residual)

    K_0 := P_00 / S;            // Kalman gain for position
    K_1 := P_10 / S;            // Kalman gain for velocity

    // State update
    X_Pos := X_Pos + K_0 * Y;
    X_Vel := X_Vel + K_1 * Y;

    // Covariance update
    Temp_00 := (1 - K_0) * P_00;
    Temp_01 := (1 - K_0) * P_01;
    Temp_10 := -K_1 * P_00 + P_10;
    Temp_11 := -K_1 * P_01 + P_11;

    P_00 := Temp_00;
    P_01 := Temp_01;
    P_10 := Temp_10;
    P_11 := Temp_11;

    // Outputs
    EstimatedPosition := X_Pos;
    EstimatedVelocity := X_Vel;
END_IF
