PROGRAM CascadeControlOilRefinery
VAR_INPUT
    Pressure_PV : REAL; // Measured pressure (bar)
    Flow_PV : REAL;     // Measured flow rate (e.g., L/min)
END_VAR

VAR_OUTPUT
    Flow_Valve_Output : REAL; // Output to drive the flow control valve (0 to 100%)
END_VAR

VAR
    // Outer loop: pressure control
    Pressure_SP : REAL := 12.0; // Setpoint pressure (bar)
    Pressure_Error : REAL;
    Flow_SP : REAL;

    // Inner loop: flow control
    Flow_Error : REAL;
    Flow_Output : REAL;

    // Controller gains
    Kp_Outer : REAL := 1.2;
    Kp_Inner : REAL := 2.5;
END_VAR

// Outer loop (pressure to flow setpoint)
Pressure_Error := Pressure_SP - Pressure_PV;
Flow_SP := Kp_Outer * Pressure_Error;

// Clamp Flow_SP to a reasonable range (e.g., 0 to 100 L/min)
IF Flow_SP < 0.0 THEN
    Flow_SP := 0.0;
ELSIF Flow_SP > 100.0 THEN
    Flow_SP := 100.0;
END_IF;

// Inner loop (flow to valve output)
Flow_Error := Flow_SP - Flow_PV;
Flow_Output := Kp_Inner * Flow_Error;

// Clamp Flow_Output to a reasonable range (e.g., 0 to 100%)
IF Flow_Output < 0.0 THEN
    Flow_Output := 0.0;
ELSIF Flow_Output > 100.0 THEN
    Flow_Output := 100.0;
END_IF;

// Assign Flow_Output to the valve control signal
Flow_Valve_Output := Flow_Output;

// Inline comments explaining the logic:
// The program implements a cascade control structure for pressure regulation in an oil refinery vessel.
// The outer loop compares the desired pressure setpoint to the actual outlet pressure and generates a flow rate setpoint.
// The inner loop adjusts the flow control valve based on flow feedback to match the flow setpoint.
// Realistic parameter values are used, including proportional gains for both loops.
// Each loop works at its natural pace: the inner loop compensates quickly for flow disturbances, while the outer loop maintains long-term pressure control with fewer oscillations.



