PROGRAM CascadeControl
TITLE 'Cascade Control - Pressure & Flow Rate'

(* Constants *)
CONST
    SAMPLE_TIME : REAL := 0.1; // Sample time in seconds (100 ms)

    // Primary Loop (Pressure) PID Gains
    KP1 : REAL := 1.5;
    KI1 : REAL := 0.2;
    KD1 : REAL := 0.05;

    // Secondary Loop (Flow Rate) PID Gains
    KP2 : REAL := 2.0;
    KI2 : REAL := 0.5;
    KD2 : REAL := 0.1;

    // Output Limits
    OP_MIN : REAL := 0.0;
    OP_MAX : REAL := 100.0;

    // Desired Pressure Setpoint
    SP1 : REAL := 100.0; // Example setpoint in appropriate units

END_CONST

VAR
    // Inputs
    PV1 : REAL; // Actual vessel pressure from ReadPressure()
    PV2 : REAL; // Actual flow rate from ReadFlowRate()

    // Outputs
    OP1 : REAL; // Output from primary loop to be used as setpoint in secondary loop
    OP2 : REAL; // Final output to SetValvePosition(OP2)

    // Internal Variables for Primary Loop
    E1 : REAL;
    Integral1 : REAL := 0.0;
    PrevError1 : REAL := 0.0;

    // Internal Variables for Secondary Loop
    E2 : REAL;
    Integral2 : REAL := 0.0;
    PrevError2 : REAL := 0.0;

END_VAR

// Primary Loop: Pressure Control
E1 := SP1 - PV1;
Integral1 := Integral1 + E1 * SAMPLE_TIME;
OP1 := KP1 * E1 + KI1 * Integral1 + KD1 * (E1 - PrevError1) / SAMPLE_TIME;

IF OP1 > OP_MAX THEN
    OP1 := OP_MAX;
ELSIF OP1 < OP_MIN THEN
    OP1 := OP_MIN;
END_IF;

PrevError1 := E1;

// Secondary Loop: Flow Rate Control
E2 := OP1 - PV2;
Integral2 := Integral2 + E2 * SAMPLE_TIME;
OP2 := KP2 * E2 + KI2 * Integral2 + KD2 * (E2 - PrevError2) / SAMPLE_TIME;

IF OP2 > OP_MAX THEN
    OP2 := OP_MAX;
ELSIF OP2 < OP_MIN THEN
    OP2 := OP_MIN;
END_IF;

PrevError2 := E2;

// Apply final output to valve position
SetValvePosition(OP2);

END_PROGRAM
