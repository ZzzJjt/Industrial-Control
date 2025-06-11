FUNCTION_BLOCK FB_HeatExchangerCascadeControl
VAR_INPUT
    // Outer loop (Temperature Control)
    Temp_SP : REAL;        // Temperature Setpoint in °C
    Temp_PV : REAL;        // Measured Temperature in °C

    // Inner loop (Flow Control)
    Flow_PV : REAL;        // Measured Flow Rate in units/min

    // Timing
    UpdateInterval : TIME := T#1s; // Interval for outer loop updates
END_VAR

VAR_OUTPUT
    Flow_Output : REAL;    // Output to drive the flow control valve
END_VAR

VAR
    // Internal Logic Variables
    Temp_Error : REAL;
    Flow_SP : REAL;

    // Proportional Gains
    Kp_Outer : REAL := 1.0;
    Kp_Inner : REAL := 2.0;

    // Timer for Outer Loop Update
    OuterLoopTimer : TON;
END_VAR

// --- STEP 1: Outer Loop - Temperature Control ---
OuterLoopTimer(IN := TRUE, PT := UpdateInterval);

IF OuterLoopTimer.Q THEN
    // Calculate the temperature error and update flow setpoint
    Temp_Error := Temp_SP - Temp_PV;
    Flow_SP := Kp_Outer * Temp_Error;

    // Reset timer to restart interval
    OuterLoopTimer(IN := FALSE);
END_IF;

// --- STEP 2: Inner Loop - Flow Control ---
// Calculate the flow error and compute output
Flow_Output := Kp_Inner * (Flow_SP - Flow_PV);

// Ensure that Flow_Output is within actuator limits if necessary
// Assuming Flow_Output drives a valve positioner, limit it between 0 and 100%
IF Flow_Output > 100.0 THEN
    Flow_Output := 100.0;
ELSIF Flow_Output < 0.0 THEN
    Flow_Output := 0.0;
END_IF;

PROGRAM PLC_PRG
VAR
    HeatCtrl : FB_HeatExchangerCascadeControl;

    // Simulated Inputs
    CurrentTemp : REAL := 80.0;
    DesiredTemp : REAL := 85.0;
    CurrentFlow : REAL := 50.0;

    // Output
    ValvePosition : REAL;
END_VAR

// Call the function block
HeatCtrl(
    Temp_SP := DesiredTemp,
    Temp_PV := CurrentTemp,
    Flow_PV := CurrentFlow,

    Flow_Output => ValvePosition
);
