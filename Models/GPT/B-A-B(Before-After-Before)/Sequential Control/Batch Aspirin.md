PROGRAM AspirinBatchControl
VAR
    // State management
    state : INT := 0;

    // Parameters
    tempReaction : REAL := 75.0;         // °C
    timeReaction : TIME := T#30m;
    tempCrystallize : REAL := 10.0;      // °C
    timeCrystallize : TIME := T#45m;
    tempDrying : REAL := 90.0;           // °C
    timeDrying : TIME := T#1h;

    // Inputs
    currentTemp : REAL;
    reactorReady : BOOL;
    dryerReady : BOOL;

    // Outputs
    heaterOn : BOOL := FALSE;
    stirrerOn : BOOL := FALSE;
    dryerOn : BOOL := FALSE;
    batchComplete : BOOL := FALSE;

    // Timer
    tHold : TON;

END_VAR

// Main process control
CASE state OF

    0: // Idle - wait for reactor to be ready
        IF reactorReady THEN
            state := 1;
        END_IF;

    1: // Heat to reaction temperature
        heaterOn := TRUE;
        IF currentTemp >= tempReaction THEN
            heaterOn := FALSE;
            stirrerOn := TRUE;
            tHold(IN := TRUE, PT := timeReaction);
            state := 2;
        END_IF;

    2: // Hold at reaction conditions
        IF tHold.Q THEN
            stirrerOn := FALSE;
            tHold(IN := FALSE);
            state := 3;
        END_IF;

    3: // Cool to crystallization temp
        IF currentTemp <= tempCrystallize THEN
            tHold(IN := TRUE, PT := timeCrystallize);
            state := 4;
        END_IF;

    4: // Hold crystallization
        IF tHold.Q THEN
            tHold(IN := FALSE);
            IF dryerReady THEN
                dryerOn := TRUE;
                state := 5;
            END_IF;
        END_IF;

    5: // Heat for drying
        IF currentTemp >= tempDrying THEN
            tHold(IN := TRUE, PT := timeDrying);
            state := 6;
        END_IF;

    6: // Hold drying
        IF tHold.Q THEN
            dryerOn := FALSE;
            tHold(IN := FALSE);
            batchComplete := TRUE;
            state := 7;
        END_IF;

    7: // Complete - wait for reset or batch report
        // Maintain completion status
END_CASE;
