VAR
    // Inputs from sensors
    FlowInput : REAL;            // SLPM
    PressureInput : REAL;        // bar

    // Setpoints and thresholds
    FlowSetpoint : REAL := 50.0; // Target flow rate
    MinPressure : REAL := 5.5;   // Safe pressure lower bound
    MaxPressure : REAL := 6.0;   // Safe pressure upper bound

    // Outputs
    FlowValveOutput : BOOL := FALSE;      // Controls airflow valve
    PressureReliefValve : BOOL := FALSE;  // Activates if pressure is out of bounds

    // Internal status flags
    FlowError : BOOL := FALSE;
    PressureError : BOOL := FALSE;
END_VAR

// === Flow Control Logic ===
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE;  // Open or increase valve
ELSIF FlowInput >= FlowSetpoint THEN
    FlowValveOutput := FALSE; // Close or hold valve
END_IF

// === Pressure Safety Monitoring ===
IF (PressureInput < MinPressure) OR (PressureInput > MaxPressure) THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE; // Engage relief logic
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF

// === Flow Fault Detection ===
IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
    FlowError := TRUE; // Large deviation detected
ELSE
    FlowError := FALSE;
END_IF
