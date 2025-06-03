METHOD PRIVATE StartHeating : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    IF currentTemp < tempSetpoint THEN
        heatingValve := TRUE;
    ELSE
        heatingValve := FALSE;
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE HoldPressure : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    IF currentTemp >= tempSetpoint AND currentPressure >= pressureSetpoint THEN
        pressureRegValve := TRUE;
        holdTimer(IN := TRUE, PT := T#20m);
    ELSE
        holdTimer(IN := FALSE);
    END_IF;

    IF holdTimer.Q THEN
        pressureRegValve := FALSE;
        done := TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE StartCooling : BOOL
VAR_OUTPUT done : BOOL;
BEGIN
    coolingValve := TRUE;
    IF currentTemp <= coolingSetpoint THEN
        coolingValve := FALSE;
        done := TRUE;
    END_IF;
END_METHOD


CASE phaseState OF
    0: // Start Heating
        IF StartHeating() THEN
            phaseState := 1;
        END_IF;

    1: // Hold Pressure & Reaction Time
        IF HoldPressure() THEN
            phaseState := 2;
        END_IF;

    2: // Start Cooling
        IF StartCooling() THEN
            phaseState := 3;
        END_IF;

    3: // Complete
        reactionComplete := TRUE;
END_CASE;
