```
FUNCTION_BLOCK KalmanFilter
VAR_INPUT
    measurement : REAL; // Measured position
    dt : REAL; // Time step between measurements
    Q : REAL; // Process noise covariance
    R : REAL; // Measurement noise covariance
    x_prev : ARRAY [1..2] OF REAL; // Previous state (position, velocity)
    P_prev : ARRAY [1..2, 1..2] OF REAL; // Previous state covariance matrix
END_VAR
VAR_OUTPUT
    x_est : ARRAY [1..2] OF REAL; // Estimated state (position, velocity)
    P_est : ARRAY [1..2, 1..2] OF REAL; // Estimated state covariance matrix
END_VAR
VAR
    F : ARRAY [1..2, 1..2] OF REAL; // State transition matrix
    H : ARRAY [1..1, 1..2] OF REAL; // Observation model
    K : ARRAY [1..2, 1..1] OF REAL; // Kalman gain
    y : REAL; // Innovation (measurement residual)
    S : REAL; // Innovation covariance
    P_pred : ARRAY [1..2, 1..2] OF REAL; // Predicted state covariance matrix
END_VAR

// Initialize matrices
F[1,1] := 1.0; F[1,2] := dt; F[2,1] := 0.0; F[2,2] := 1.0;
H[1,1] := 1.0; H[1,2] := 0.0;

// Prediction step
x_pred := F * x_prev;
P_pred := F * P_prev * TRANSPOSE(F) + Q;

// Update step
y := measurement - H * x_pred;
S := H * P_pred * TRANSPOSE(H) + R;
K := P_pred * TRANSPOSE(H) / S;
x_est := x_pred + K * y;
P_est := (IDENTITY(2) - K * H) * P_pred;

RETURN;

END_FUNCTION_BLOCK


PROGRAM ExampleProgram
VAR
    measuredPosition : REAL := 0.0; // Measured position
    deltaTime : REAL := 1.0; // Time step between measurements
    processNoise : REAL := 0.1; // Process noise covariance
    measurementNoise : REAL := 1.0; // Measurement noise covariance
    prevState : ARRAY [1..2] OF REAL := {0.0, 0.0}; // Initial state (position, velocity)
    prevCovariance : ARRAY [1..2, 1..2] OF REAL := {{1.0, 0.0}, {0.0, 1.0}}; // Initial state covariance
    estimatedState : ARRAY [1..2] OF REAL; // Estimated state (position, velocity)
    estimatedCovariance : ARRAY [1..2, 1..2] OF REAL; // Estimated state covariance
BEGIN
    // Simulate a new measurement (for demonstration purposes)
    measuredPosition := /* New measured position */;
    
    // Call the KalmanFilter function block
    KalmanFilter(measurement:=measuredPosition, dt:=deltaTime, Q:=processNoise, R:=measurementNoise,
                 x_prev:=prevState, P_prev:=prevCovariance, x_est:=estimatedState, P_est:=estimatedCovariance);
    
    // Update the previous state and covariance for the next iteration
    prevState := estimatedState;
    prevCovariance := estimatedCovariance;
    
    // Optionally, log or use the estimated state
END_PROGRAM
```
