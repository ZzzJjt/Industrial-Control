PROGRAM PneumaticSystemControl
VAR_INPUT
    FlowInput : REAL; // Current airflow rate in SLPM
    PressureInput : REAL; // Current pressure in bar
END_VAR

VAR_OUTPUT
    FlowValveOutput : BOOL := FALSE; // Output to control the flow valve
    PressureReliefValve : BOOL := FALSE; // Output to control the pressure relief valve
END_VAR

VAR
    FlowSetpoint : REAL := 50.0; // Desired flow rate setpoint in SLPM
    MinPressure : REAL := 5.5; // Minimum safe pressure in bar
    MaxPressure : REAL := 6.0; // Maximum safe pressure in bar
    FlowError : BOOL := FALSE; // Flag indicating flow error
    PressureError : BOOL := FALSE; // Flag indicating pressure error
    LastTime : TIME; // Variable to store the last execution time
    CycleTime : TIME := T#100ms; // Control cycle time
END_VAR

// Check if the control cycle time has elapsed
IF TONR(T:=CycleTime, ET=>LastTime).Q THEN
    LastTime := TONR(T:=CycleTime, ET=>LastTime).ET;

    // Flow control logic
    IF FlowInput < FlowSetpoint THEN
        FlowValveOutput := TRUE;
    ELSE
        FlowValveOutput := FALSE;
    END_IF;

    // Pressure check logic
    IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
        PressureError := TRUE;
        PressureReliefValve := TRUE;
    ELSE
        PressureError := FALSE;
        PressureReliefValve := FALSE;
    END_IF;

    // Flow deviation check
    IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
        FlowError := TRUE;
    ELSE
        FlowError := FALSE;
    END_IF;
END_IF;

// Additional comments for clarity
// - Maintain airflow stability at 50 SLPM by controlling the flow valve.
// - Keep system pressure safely between 5.5 and 6.0 bar using the pressure relief valve.
// - Detect and report anomalies in flow or pressure.
// - Run reliably in a real-time industrial control loop with a 100 ms cycle time.



