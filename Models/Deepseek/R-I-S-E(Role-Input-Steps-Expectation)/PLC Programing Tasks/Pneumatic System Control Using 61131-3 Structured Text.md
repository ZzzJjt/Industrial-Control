FUNCTION_BLOCK FB_PneumaticControl
VAR_INPUT
    // Sensor Inputs
    FlowInput : REAL;         // Current flow rate in SLPM
    PressureInput : REAL;     // Current pressure in bar

    // Control Parameters
    FlowSetpoint : REAL := 50.0;      // Target flow rate
    MinPressure : REAL := 5.5;       // Minimum safe pressure
    MaxPressure : REAL := 6.0;       // Maximum safe pressure
    FlowTolerance : REAL := 5.0;     // Acceptable deviation from setpoint
END_VAR

VAR_OUTPUT
    // Actuator Outputs
    FlowValveOutput : BOOL := FALSE;
    PressureReliefValve : BOOL := FALSE;

    // Diagnostic Flags
    FlowError : BOOL := FALSE;
    PressureError : BOOL := FALSE;
END_VAR

// --- STEP 1: Flow Control Logic ---
IF FlowInput < (FlowSetpoint - FlowTolerance) THEN
    // Too low — open valve to increase airflow
    FlowValveOutput := TRUE;
ELSIF FlowInput > (FlowSetpoint + FlowTolerance) THEN
    // Too high — close valve to reduce airflow
    FlowValveOutput := FALSE;
ELSE
    // Within tolerance — maintain current state
    // Optional: keep valve open if near lower limit, or closed if near upper
    FlowValveOutput := FlowValveOutput; // Hold last state
END_IF;

// --- STEP 2: Pressure Safety Monitoring ---
IF (PressureInput < MinPressure) OR (PressureInput > MaxPressure) THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE;
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF;

// --- STEP 3: Flow Deviation Flagging ---
IF ABS(FlowInput - FlowSetpoint) > FlowTolerance THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF;

PROGRAM PLC_PRG
VAR
    PneumaticCtrl : FB_PneumaticControl;

    // Simulated Inputs
    CurrentFlow : REAL := 48.0;
    CurrentPressure : REAL := 5.7;

    // Output Flags
    ValveState : BOOL;
    ReliefValveState : BOOL;
    FlowAlarm : BOOL;
    PressureAlarm : BOOL;
END_VAR

// Call the function block
PneumaticCtrl(
    FlowInput := CurrentFlow,
    PressureInput := CurrentPressure,

    FlowSetpoint := 50.0,
    MinPressure := 5.5,
    MaxPressure := 6.0,
    FlowTolerance := 5.0,

    FlowValveOutput => ValveState,
    PressureReliefValve => ReliefValveState,
    FlowError => FlowAlarm,
    PressureError => PressureAlarm
);
