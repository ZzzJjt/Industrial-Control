PROGRAM PneumaticControl
VAR
    // Inputs
    FlowInput           : REAL;     // Measured flow in SLPM
    PressureInput       : REAL;     // Measured pressure in bar

    // Setpoints and thresholds
    FlowSetpoint        : REAL := 50.0;   // Desired flow rate in SLPM
    MinPressure         : REAL := 5.5;    // Lower safe pressure threshold
    MaxPressure         : REAL := 6.0;    // Upper safe pressure threshold
    MaxFlowDeviation    : REAL := 5.0;    // Acceptable deviation in SLPM

    // Outputs
    FlowValveOutput     : BOOL := FALSE;  // Controls flow actuator
    PressureReliefValve : BOOL := FALSE;  // Safety valve to vent excess pressure

    // Fault flags
    FlowError           : BOOL := FALSE;  // Flow rate abnormal
    PressureError       : BOOL := FALSE;  // Over/under pressure

    // Optional: System health output
    SystemOK            : BOOL := TRUE;   // General system status
END_VAR

// === Flow Control ===
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE;  // Open valve to increase flow
ELSIF FlowInput >= FlowSetpoint THEN
    FlowValveOutput := FALSE; // Close valve or maintain
END_IF;

// === Flow Deviation Fault Check ===
IF ABS(FlowInput - FlowSetpoint) > MaxFlowDeviation THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF;

// === Pressure Safety Logic ===
IF (PressureInput < MinPressure) OR (PressureInput > MaxPressure) THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE;
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF;

// === System Health Status ===
SystemOK := NOT (FlowError OR PressureError);
