PROGRAM PneumaticControl
VAR
    // Inputs
    FlowInput       : REAL;   // Current flow rate (SLPM)
    PressureInput   : REAL;   // Current system pressure (bar)

    // Setpoints and limits
    FlowSetpoint    : REAL := 50.0; // Target flow rate
    MinPressure     : REAL := 5.5;  // Lower pressure threshold
    MaxPressure     : REAL := 6.0;  // Upper pressure threshold

    // Outputs
    FlowValveOutput     : BOOL := FALSE;  // Open valve to increase flow
    PressureReliefValve : BOOL := FALSE;  // Vent pressure if out of range

    // Fault indicators
    FlowError       : BOOL := FALSE;
    PressureError   : BOOL := FALSE;
END_VAR

// --- Flow Control Logic ---
// Open valve if flow is below target
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE;
ELSE
    FlowValveOutput := FALSE;
END_IF

// --- Pressure Monitoring and Safety Logic ---
// Activate relief valve and raise alarm if pressure is out of range
IF (PressureInput < MinPressure) OR (PressureInput > MaxPressure) THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE;
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF

// --- Flow Deviation Fault Detection ---
// Raise error if deviation exceeds ±5.0 SLPM
IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF
