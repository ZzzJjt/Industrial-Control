PROGRAM UreaReactionControl
VAR
    // Outputs
    AmmoniaValveCmd     : BOOL := FALSE;
    CO2ValveCmd         : BOOL := FALSE;
    Finished            : BOOL := FALSE;

    // Inputs
    AmmoniaValveStatus  : BOOL;
    CO2ValveStatus      : BOOL;
    Pressure            : REAL;
    Temperature         : REAL;

    // Parameters
    TargetPressure      : REAL := 150.0;    // bar
    PressureTolerance   : REAL := 5.0;
    TargetTemperature   : REAL := 180.0;    // °C
    TempTolerance       : REAL := 3.0;
    ReactionTime        : TIME := T#10m;    // Reaction duration

    // Internal state
    Step_LoadingDone    : BOOL := FALSE;
    Step_ReactionDone   : BOOL := FALSE;
    TimerRunning        : BOOL := FALSE;
    StartTime           : TIME;
    CurrentTime         : TIME;             // Should be assigned system time externally
END_VAR

// Step 1: Load reactants
IF NOT Step_LoadingDone THEN
    AmmoniaValveCmd := TRUE;
    CO2ValveCmd := TRUE;

    IF AmmoniaValveStatus AND CO2ValveStatus THEN
        Step_LoadingDone := TRUE;
    END_IF
END_IF

// Step 2: Check pressure and temperature
IF Step_LoadingDone AND NOT Step_ReactionDone THEN
    IF ABS(Pressure - TargetPressure) <= PressureTolerance AND
       ABS(Temperature - TargetTemperature) <= TempTolerance THEN

        IF NOT TimerRunning THEN
            StartTime := CurrentTime;
            TimerRunning := TRUE;
        END_IF

        // Step 3: Time the reaction
        IF TimerRunning AND (CurrentTime - StartTime >= ReactionTime) THEN
            Step_ReactionDone := TRUE;
            TimerRunning := FALSE;
        END_IF
    ELSE
        // Reset timer if conditions go out of range
        TimerRunning := FALSE;
    END_IF
END_IF

// Step 4: Finish sequence
IF Step_ReactionDone THEN
    AmmoniaValveCmd := FALSE;
    CO2ValveCmd := FALSE;
    Finished := TRUE;
END_IF
