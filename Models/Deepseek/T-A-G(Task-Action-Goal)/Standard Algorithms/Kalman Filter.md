FUNCTION_BLOCK KalmanFilterFunctionBlock
VAR_INPUT
    MeasuredPosition : REAL; // Measured position from sensor
    Q : REAL;                // Process noise covariance
    R : REAL;                // Measurement noise covariance
    DeltaT : REAL;           // Sample time
    Reset : BOOL;            // Signal to reinitialize state
END_VAR

VAR_OUTPUT
    EstimatedPosition : REAL; // Estimated position
    EstimatedVelocity : REAL; // Estimated velocity
END_VAR

VAR
    PreviousPosition : REAL := 0.0; // Previous position estimate
    PreviousVelocity : REAL := 0.0; // Previous velocity estimate
    ErrorCovariance : REAL := 1.0;  // Initial error covariance
    KalmanGain : REAL;              // Kalman gain
    PredictedPosition : REAL;        // Predicted position
    PredictedVelocity : REAL;        // Predicted velocity
    PredictedErrorCovariance : REAL; // Predicted error covariance
END_VAR

METHOD Execute;
BEGIN
    // Reset the filter if requested
    IF Reset THEN
        EstimatedPosition := 0.0;
        EstimatedVelocity := 0.0;
        PreviousPosition := 0.0;
        PreviousVelocity := 0.0;
        ErrorCovariance := 1.0;
    END_IF;

    // Prediction step
    PredictedPosition := PreviousPosition + PreviousVelocity * DeltaT;
    PredictedVelocity := PreviousVelocity;
    PredictedErrorCovariance := ErrorCovariance + Q;

    // Update step
    KalmanGain := PredictedErrorCovariance / (PredictedErrorCovariance + R);
    EstimatedPosition := PredictedPosition + KalmanGain * (MeasuredPosition - PredictedPosition);
    EstimatedVelocity := PredictedVelocity + KalmanGain * ((MeasuredPosition - PredictedPosition) / DeltaT);
    ErrorCovariance := (1 - KalmanGain) * PredictedErrorCovariance;

    // Update previous states
    PreviousPosition := EstimatedPosition;
    PreviousVelocity := EstimatedVelocity;
END_METHOD



