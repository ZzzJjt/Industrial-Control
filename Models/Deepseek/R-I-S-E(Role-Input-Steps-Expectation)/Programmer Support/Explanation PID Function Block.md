FUNCTION_BLOCK PID_SIMPLE
VAR_INPUT
    SP      : REAL;        // Setpoint
    PV      : REAL;        // Process Variable
    K_P     : REAL;        // Proportional gain
    K_I     : REAL;        // Integral gain (1/min)
    K_D     : REAL;        // Derivative gain (min)
    T_SAMPLE: TIME := T#100ms; // Sampling time
    MANUAL  : BOOL := FALSE;   // Manual mode (if TRUE, OUT is not computed)
    OUT_MAN : REAL := 0.0;     // Manual output value
    LIMIT_LO: REAL := 0.0;     // Lower output limit
    LIMIT_HI: REAL := 100.0;   // Upper output limit
END_VAR

VAR_OUTPUT
    OUT     : REAL;        // Calculated control output
    ERROR   : REAL;        // Error = SP - PV
    SATURATED: BOOL;       // TRUE if output is at LIMIT_HI or LIMIT_LO
END_VAR

VAR
    // Internal variables managed by the block
    last_error   : REAL := 0.0;
    integral_term: REAL := 0.0;
    derivative_term: REAL := 0.0;
    prev_pv      : REAL := 0.0;
END_VAR

PROGRAM PLC_PRG
VAR
    MyPID       : PID_SIMPLE;

    // Process inputs
    Temperature_SP : REAL := 75.0;  // °C target
    Temperature_PV : REAL := 72.3;  // Measured from sensor

    // Control output
    Heater_Output : REAL;          // Sent to analog output or PWM
END_VAR

MyPID(
    SP        := Temperature_SP,
    PV        := Temperature_PV,
    K_P       := 5.0,       // Tune based on process
    K_I       := 0.2,       // Tune based on process
    K_D       := 0.5,       // Optional, often set low or zero
    T_SAMPLE  := T#100ms,   // Must match scan rate
    MANUAL    := FALSE,     // Automatic mode
    LIMIT_LO  := 0.0,        // Minimum heater output
    LIMIT_HI  := 100.0,      // Maximum heater output

    OUT       => Heater_Output,
    ERROR     => ,           // Optional connection
    SATURATED =>            // Optional status flag
);
