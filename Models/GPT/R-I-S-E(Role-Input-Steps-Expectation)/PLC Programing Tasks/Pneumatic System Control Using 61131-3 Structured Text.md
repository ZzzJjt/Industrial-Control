FUNCTION_BLOCK PneumaticController
VAR_INPUT
    FlowInput       : REAL; // Measured flow rate (SLPM)
    PressureInput   : REAL; // Measured pressure (bar)
END_VAR

VAR_OUTPUT
    FlowValveOutput     : BOOL := FALSE; // Air valve control
    PressureReliefValve : BOOL := FALSE; // Pressure relief control
    FlowError           : BOOL := FALSE; // Diagnostic flag
    PressureError       : BOOL := FALSE; // Diagnostic flag
END_VAR

VAR CONSTANT
    FlowSetpoint    : REAL := 50.0; // Target flow rate (SLPM)
    FlowTolerance   : REAL := 5.0;  // Allowed flow deviation (±)
    MinPressure     : REAL := 5.5;  // Minimum safe pressure
    MaxPressure     : REAL := 6.0;  // Maximum safe pressure
END_VAR

// --- Flow Control Logic ---
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE;
ELSE
    FlowValveOutput := FALSE;
END_IF

// --- Pressure Safety Check ---
IF (PressureInput < MinPressure) OR (PressureInput > MaxPressure) THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE;
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF

// --- Flow Error Monitoring ---
IF ABS(FlowInput - FlowSetpoint) > FlowTolerance THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF
