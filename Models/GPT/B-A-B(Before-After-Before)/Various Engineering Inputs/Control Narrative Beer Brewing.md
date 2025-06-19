(*
Function Block: FB_LauteringControl
Description: Implements lautering logic for brewery automation including rake control,
             flow ramp-up, turbidity monitoring, and sparge water handling.
*)

FUNCTION_BLOCK FB_LauteringControl
VAR_INPUT
    StartLautering      : BOOL;
    Turbidity           : REAL;    // NTU
    LauterTunLevel      : REAL;    // %
    SpargeWaterTemp     : REAL;    // °C
    FlowRate            : REAL;    // L/min
    FlowSetpoint        : REAL := 1.8; // L/min (target)
    MaxTurbidity        : REAL := 80.0; // NTU
    MinLevel            : REAL := 50.0; // %
    MaxLevel            : REAL := 80.0; // %
END_VAR

VAR_OUTPUT
    RakeMotorOn         : BOOL;
    SpargeValveOpen     : BOOL;
    WortToTankValve     : BOOL;
    WortToWasteValve    : BOOL;
    LauteringDone       : BOOL;
END_VAR

VAR
    FlowRampStep        : REAL := 0.1; // L/min per cycle
    FlowControlTimer    : TON;
    RampUpTimer         : TON;
    FlowReady           : BOOL;
END_VAR

// Flow ramp-up logic
IF StartLautering AND NOT LauteringDone THEN
    RakeMotorOn := TRUE;

    // Ramp up flow to setpoint
    IF FlowRate < FlowSetpoint THEN
        RampUpTimer(IN := TRUE, PT := T#5S);
        IF RampUpTimer.Q THEN
            FlowRate := FlowRate + FlowRampStep;
            RampUpTimer(IN := FALSE);
        END_IF
        FlowReady := FALSE;
    ELSE
        FlowReady := TRUE;
    END_IF

    // Open valves based on turbidity and tank level
    IF FlowReady THEN
        IF Turbidity > MaxTurbidity THEN
            WortToTankValve := FALSE;
            WortToWasteValve := TRUE;
        ELSE
            WortToTankValve := TRUE;
            WortToWasteValve := FALSE;
        END_IF

        // Stop sparge if level too low
        IF LauterTunLevel < MinLevel THEN
            SpargeValveOpen := FALSE;
        ELSIF LauterTunLevel > MaxLevel THEN
            SpargeValveOpen := FALSE;
        ELSE
            SpargeValveOpen := TRUE;
        END_IF
    END_IF

    // Check completion criteria
    IF Turbidity > MaxTurbidity AND LauterTunLevel < MinLevel THEN
        LauteringDone := TRUE;
        RakeMotorOn := FALSE;
        SpargeValveOpen := FALSE;
        WortToTankValve := FALSE;
        WortToWasteValve := FALSE;
    END_IF
ELSE
    // Reset all outputs
    RakeMotorOn := FALSE;
    SpargeValveOpen := FALSE;
    WortToTankValve := FALSE;
    WortToWasteValve := FALSE;
    LauteringDone := FALSE;
END_IF
