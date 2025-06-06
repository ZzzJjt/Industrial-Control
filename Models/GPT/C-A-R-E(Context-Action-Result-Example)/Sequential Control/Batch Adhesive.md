FUNCTION_BLOCK FB_ReactionStep
VAR_INPUT
    Start           : BOOL;
    TargetTemp      : REAL := 85.0;
    MixSpeed        : INT := 600;
    HeatHoldTime    : TIME := T#10m;
    MixTime         : TIME := T#15m;
    HoldTime        : TIME := T#20m;
    CurrentTemp     : REAL;
END_VAR

VAR_OUTPUT
    Done            : BOOL;
    StepStatus      : STRING[50];
END_VAR

VAR
    Phase           : INT := 0;
    HeatingTimer    : TON;
    MixingTimer     : TON;
    HoldingTimer    : TON;
END_VAR
