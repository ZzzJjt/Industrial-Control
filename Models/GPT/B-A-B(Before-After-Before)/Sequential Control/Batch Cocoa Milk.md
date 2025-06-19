PROGRAM CocoaMilkBatchControl
VAR
    // State logic
    state : INT := 0;

    // Ingredient targets
    milkTarget : REAL := 60.0;
    waterTarget : REAL := 20.0;
    sugarTarget : REAL := 15.0;
    cocoaTarget : REAL := 5.0;

    // Process parameters
    mixTemp : REAL := 70.0;
    stirSpeed : INT := 200;
    mixDuration : TIME := T#10m;

    // Sensor inputs
    currentTemp : REAL;
    milkLevel : REAL;
    waterLevel : REAL;
    sugarLevel : REAL;
    cocoaLevel : REAL;

    // Actuators
    valveMilk : BOOL := FALSE;
    valveWater : BOOL := FALSE;
    valveSugar : BOOL := FALSE;
    valveCocoa : BOOL := FALSE;
    heater : BOOL := FALSE;
    stirrer : BOOL := FALSE;

    // Timer
    tMix : TON;

    // Output
    batchComplete : BOOL := FALSE;
END_VAR

// Main process control
CASE state OF

    0: // Idle
        IF milkLevel < milkTarget THEN
            valveMilk := TRUE;
        END_IF;
        IF waterLevel < waterTarget THEN
            valveWater := TRUE;
        END_IF;
        IF sugarLevel < sugarTarget THEN
            valveSugar := TRUE;
        END_IF;
        IF cocoaLevel < cocoaTarget THEN
            valveCocoa := TRUE;
        END_IF;

        IF milkLevel >= milkTarget AND waterLevel >= waterTarget AND
           sugarLevel >= sugarTarget AND cocoaLevel >= cocoaTarget THEN
            valveMilk := FALSE;
            valveWater := FALSE;
            valveSugar := FALSE;
            valveCocoa := FALSE;
            state := 1;
        END_IF;

    1: // Heating
        heater := TRUE;
        IF currentTemp >= mixTemp THEN
            heater := FALSE;
            state := 2;
        END_IF;

    2: // Mixing
        stirrer := TRUE;
        tMix(IN := TRUE, PT := mixDuration);
        IF tMix.Q THEN
            stirrer := FALSE;
            tMix(IN := FALSE);
            state := 3;
        END_IF;

    3: // Complete
        batchComplete := TRUE;

END_CASE;
