PROGRAM PVCBatchControl
VAR
    state : INT := 0;
    timer : TON;
    currentTemp : REAL;
    currentPressure : REAL;

    // Setpoints
    polymerTempLow : REAL := 55.0;
    polymerTempHigh : REAL := 60.0;
    endPressure : REAL := 1.2; // bar

    // Outputs
    reactorVacuum : BOOL := FALSE;
    waterValve : BOOL := FALSE;
    surfactantValve : BOOL := FALSE;
    catalystValve : BOOL := FALSE;
    vcmValve : BOOL := FALSE;
    heater : BOOL := FALSE;
    agitator : BOOL := FALSE;
    dryer : BOOL := FALSE;
    batchComplete : BOOL := FALSE;
END_VAR

CASE state OF

    0: // Evacuate reactor
        reactorVacuum := TRUE;
        timer(IN := TRUE, PT := T#5m); // Evacuate for 5 minutes
        IF timer.Q THEN
            reactorVacuum := FALSE;
            timer(IN := FALSE);
            state := 1;
        END_IF;

    1: // Add demineralized water
        waterValve := TRUE;
        timer(IN := TRUE, PT := T#2m); // 2 min dosing
        IF timer.Q THEN
            waterValve := FALSE;
            timer(IN := FALSE);
            state := 2;
        END_IF;

    2: // Add surfactant
        surfactantValve := TRUE;
        timer(IN := TRUE, PT := T#30s);
        IF timer.Q THEN
            surfactantValve := FALSE;
            timer(IN := FALSE);
            state := 3;
        END_IF;

    3: // Add catalyst
        catalystValve := TRUE;
        timer(IN := TRUE, PT := T#30s);
        IF timer.Q THEN
            catalystValve := FALSE;
            timer(IN := FALSE);
            state := 4;
        END_IF;

    4: // Add VCM
        vcmValve := TRUE;
        timer(IN := TRUE, PT := T#3m);
        IF timer.Q THEN
            vcmValve := FALSE;
            timer(IN := FALSE);
            state := 5;
        END_IF;

    5: // Polymerization
        heater := TRUE;
        agitator := TRUE;
        IF currentTemp >= polymerTempLow AND currentTemp <= polymerTempHigh THEN
            IF currentPressure <= endPressure THEN
                heater := FALSE;
                agitator := FALSE;
                state := 6;
            END_IF;
        END_IF;

    6: // Decovar
        reactorVacuum := TRUE;
        timer(IN := TRUE, PT := T#2m);
        IF timer.Q THEN
            reactorVacuum := FALSE;
            timer(IN := FALSE);
            state := 7;
        END_IF;

    7: // Dry
        dryer := TRUE;
        timer(IN := TRUE, PT := T#30m);
        IF timer.Q THEN
            dryer := FALSE;
            timer(IN := FALSE);
            batchComplete := TRUE;
            state := 8;
        END_IF;

    8: // Idle / Done
        // Hold state
END_CASE;
