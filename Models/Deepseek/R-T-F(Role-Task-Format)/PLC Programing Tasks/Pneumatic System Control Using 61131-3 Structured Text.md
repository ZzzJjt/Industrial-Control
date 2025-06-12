PROGRAM PneumaticSystemControl
VAR
    FlowInput : REAL; // Sensor input for flow rate (SLPM)
    PressureInput : REAL; // Sensor input for pressure (bar)
    FlowSetpoint : REAL := 50.0; // Setpoint for flow rate (SLPM)
    MinPressure : REAL := 5.5; // Minimum acceptable pressure (bar)
    MaxPressure : REAL := 6.0; // Maximum acceptable pressure (bar)

    FlowValveOutput : BOOL := FALSE; // Output to control the flow valve
    PressureReliefValve : BOOL := FALSE; // Output to control the pressure relief valve
    FlowError : BOOL := FALSE; // Flag indicating flow error
    PressureError : BOOL := FALSE; // Flag indicating pressure error
END_VAR

// Flow control
IF FlowInput < FlowSetpoint THEN
    FlowValveOutput := TRUE; // Open the valve to increase flow
ELSE
    FlowValveOutput := FALSE; // Hold or close the valve
END_IF;

// Pressure safety logic
IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
    PressureError := TRUE;
    PressureReliefValve := TRUE; // Activate the pressure relief valve
ELSE
    PressureError := FALSE;
    PressureReliefValve := FALSE; // Deactivate the pressure relief valve
END_IF;

// Flow deviation detection
IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
    FlowError := TRUE; // Set flow error flag if deviation exceeds 5.0 SLPM
ELSE
    FlowError := FALSE; // Clear flow error flag
END_IF;
