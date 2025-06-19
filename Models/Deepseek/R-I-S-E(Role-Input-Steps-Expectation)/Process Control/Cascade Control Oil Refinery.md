FUNCTION_BLOCK FB_PressureCascadeControl
VAR_INPUT
    // Outer Loop Inputs - Pressure Control
    Pressure_SP : REAL := 12.0;      // Vessel pressure setpoint (bar)
    Pressure_PV : REAL;              // Measured vessel pressure (bar)

    // Inner Loop Inputs - Flow Control
    Flow_PV : REAL;                  // Measured oil flow rate (units/min)

    // Timing Parameters
    OuterLoopInterval : TIME := T#2s; // Update interval for outer loop
END_VAR

VAR_OUTPUT
    // Output to actuator
    Flow_Output : REAL;              // Command to control valve (0–100%)
END_VAR

VAR
    // Internal Logic Variables
    Pressure_Error : REAL;
    Flow_SP : REAL;

    Flow_Error : REAL;

    // Controller Gains
    Kp_Outer : REAL := 1.2;          // Proportional gain for pressure loop
    Kp_Inner : REAL := 2.5;          // Proportional gain for flow loop

    // Timer for outer loop execution
    OuterTimer : TON;
END_VAR

// --- STEP 1: Outer Loop - Pressure Controller ---
OuterTimer(IN := TRUE, PT := OuterLoopInterval);

IF OuterTimer.Q THEN
    // Calculate pressure error
    Pressure_Error := Pressure_SP - Pressure_PV;

    // Compute flow setpoint using proportional gain
    Flow_SP := Kp_Outer * Pressure_Error;

    // Reset timer for next update
    OuterTimer(IN := FALSE);
END_IF;

// --- STEP 2: Inner Loop - Flow Controller ---
// Calculate flow error
Flow_Error := Flow_SP - Flow_PV;

// Apply inner loop proportional gain
Flow_Output := Kp_Inner * Flow_Error;

// Clamp output to valid range (e.g., 0–100% for valve position)
IF Flow_Output > 100.0 THEN
    Flow_Output := 100.0;
ELSIF Flow_Output < 0.0 THEN
    Flow_Output := 0.0;
END_IF;

PROGRAM PLC_PRG
VAR
    CascadeCtrl : FB_PressureCascadeControl;

    // Simulated Inputs
    CurrentPressure : REAL := 11.8;
    DesiredPressure : REAL := 12.0;
    CurrentFlowRate : REAL := 45.0;

    // Output
    ValveCommand : REAL;
END_VAR

// Call the function block
CascadeCtrl(
    Pressure_SP := DesiredPressure,
    Pressure_PV := CurrentPressure,
    Flow_PV := CurrentFlowRate,

    Flow_Output => ValveCommand
);
