PROGRAM PolyethyleneBatchControl
VAR
    phaseState : INT := 0;
    phaseTimer : TON;

    currentTemp : REAL;
    currentPressure : REAL;

    phaseComplete : BOOL := FALSE;
    heater : BOOL := FALSE;
    agitator : BOOL := FALSE;
    conveyor : BOOL := FALSE;
    pelletizer : BOOL := FALSE;
    dryer : BOOL := FALSE;
    packager : BOOL := FALSE;

    batchComplete : BOOL := FALSE;
END_VAR

METHOD PRIVATE ExecuteRawMaterialPreparation : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    heater := TRUE;
    agitator := TRUE;
    phaseTimer(IN := TRUE, PT := T#5m); // Heat & mix for 5 minutes
    IF phaseTimer.Q THEN
        heater := FALSE;
        agitator := FALSE;
        phaseTimer(IN := FALSE);
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE ExecutePolymerization : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    heater := TRUE;
    IF currentTemp >= 160.0 AND currentPressure >= 25.0 THEN
        phaseTimer(IN := TRUE, PT := T#30m);
    END_IF;

    IF phaseTimer.Q THEN
        heater := FALSE;
        phaseTimer(IN := FALSE);
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE ExecuteQuenching : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    phaseTimer(IN := TRUE, PT := T#10m);
    IF phaseTimer.Q THEN
        phaseTimer(IN := FALSE);
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE ExecuteDrying : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    dryer := TRUE;
    phaseTimer(IN := TRUE, PT := T#1h);
    IF phaseTimer.Q THEN
        dryer := FALSE;
        phaseTimer(IN := FALSE);
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE ExecutePelletizing : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    pelletizer := TRUE;
    phaseTimer(IN := TRUE, PT := T#45m);
    IF phaseTimer.Q THEN
        pelletizer := FALSE;
        phaseTimer(IN := FALSE);
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE ExecuteQualityControl : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    phaseTimer(IN := TRUE, PT := T#15m);
    IF phaseTimer.Q THEN
        phaseTimer(IN := FALSE);
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE ExecutePackaging : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    packager := TRUE;
    phaseTimer(IN := TRUE, PT := T#30m);
    IF phaseTimer.Q THEN
        packager := FALSE;
        phaseTimer(IN := FALSE);
        done := TRUE;
    END_IF;
END_METHOD
