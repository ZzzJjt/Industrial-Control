VAR
    // Inputs
    FlowInput           : REAL;      // Current airflow rate in SLPM
    PressureInput       : REAL;      // Current pressure in bar

    // Constants
    FlowSetpoint        : REAL := 50.0;   // Desired flow rate
    MinPressure         : REAL := 5.5;    // Lower safe pressure limit
    MaxPressure         : REAL := 6.0;    // Upper safe pressure limit
    MaxFlowDeviation    : REAL := 5.0;    // Allowed deviation in flow

    // Outputs
    FlowValveOutput     : BOOL := FALSE;  // Controls the flow valve
    PressureReliefValve : BOOL := FALSE;  // Activates pressure relief

    // Alarms / Flags
    FlowError           : BOOL := FALSE;
    PressureError       : BOOL := FALSE;
END_VAR

//--------------------------//
// Flow Control Logic       //
//--------------------------//
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE;   // Open valve to increase flow
ELSE
    FlowValveOutput := FALSE;  // Hold or close valve
END_IF

//--------------------------//
// Pressure Protection Logic//
//--------------------------//
IF (PressureInput < MinPressure) OR (PressureInput > MaxPressure) THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE;   // Trigger pressure relief mechanism
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE;
END_IF

//------------------------------------//
// Flow Deviation Fault Detection     //
//------------------------------------//
IF ABS(FlowInput - FlowSetpoint) > MaxFlowDeviation THEN
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF
