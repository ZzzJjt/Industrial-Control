FUNCTION_BLOCK FB_GasFlowRampDown
VAR_INPUT
    StartFlow : REAL;         // Initial gas flow in Nm³/h
    EndFlow   : REAL;         // Target gas flow
    Duration  : TIME;         // Total ramp time
    Start     : BOOL;         // Trigger input
END_VAR
VAR_OUTPUT
    FlowSetpoint : REAL;      // Output setpoint
    Done         : BOOL;
END_VAR
VAR
    RampTimer    : TON;
    TimeElapsed  : TIME;
    Slope        : REAL;
END_VAR

IF Start AND NOT Done THEN
    RampTimer(IN := TRUE, PT := Duration);
    TimeElapsed := RampTimer.ET;

    Slope := (EndFlow - StartFlow) / TO_REAL(TO_TIME(Duration) / T#1s);
    FlowSetpoint := StartFlow + Slope * TO_REAL(TimeElapsed / T#1s);

    IF RampTimer.Q THEN
        RampTimer(IN := FALSE);
        FlowSetpoint := EndFlow;
        Done := TRUE;
    END_IF
END_IF

FUNCTION_BLOCK FB_OxygenControl
VAR_INPUT
    FuelFlow        : REAL;     // Current gas flow in Nm³/h
    TargetRatio     : REAL;     // Target fuel-to-air ratio (e.g., 1:2.5)
END_VAR
VAR_OUTPUT
    OxygenSetpoint  : REAL;     // Output oxygen setpoint
END_VAR

OxygenSetpoint := FuelFlow * TargetRatio;

VAR
    Step               : INT := 4;
    FurnaceTemp        : REAL;
    FuelFlowCurrent    : REAL := 100.0;

    RampFlow           : FB_GasFlowRampDown;
    OxygenCtrl         : FB_OxygenControl;

    FinalGasFlow       : REAL;
    OxygenSetpoint     : REAL;

    ShutdownTimer      : TON;
    StepDone           : BOOL;
END_VAR

CASE Step OF

4: // Lower Furnace Temperature
    IF FurnaceTemp > 400.0 THEN
        RampFlow(StartFlow := 100.0, EndFlow := 0.0, Duration := T#12h, Start := TRUE);
        FinalGasFlow := RampFlow.FlowSetpoint;
        IF RampFlow.Done THEN
            Step := 5;
        END_IF
    ELSE
        Step := 5;
    END_IF

5: // Adjust Oxygen Level to Maintain 1:2.5 Ratio
    FuelFlowCurrent := FinalGasFlow;
    OxygenCtrl(FuelFlow := FuelFlowCurrent, TargetRatio := 2.5);
    OxygenSetpoint := OxygenCtrl.OxygenSetpoint;

    IF OxygenSetpoint <= 50.0 THEN // Arbitrary threshold
        Step := 6;
    END_IF

6: // Wait for Burnout Period (e.g., 30 min hold)
    ShutdownTimer(IN := TRUE, PT := T#30m);
    IF ShutdownTimer.Q THEN
        ShutdownTimer(IN := FALSE);
        StepDone := TRUE;
    END_IF

END_CASE
