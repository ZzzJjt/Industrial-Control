PROGRAM PolyethyleneBatchControl
VAR
    state : INT := 0;
    timer : TON;
    stepStartTime : TIME;
    
    // Process Parameters
    rawMatPrepTemp : REAL := 70.0;
    rawMatPrepPressure : REAL := 1.0;
    polymerizationTemp : REAL := 150.0;
    polymerizationPressure : REAL := 30.0;
    quenchingTemp : REAL := 25.0;
    quenchingPressure : REAL := 5.0;
    dryingTemp : REAL := 80.0;
    pelletizingTemp : REAL := 150.0;
    qualityControlTemp : REAL := 25.0;
    packagingStorageTemp : REAL := 20.0;

    stepDurations : ARRAY[1..7] OF TIME := [
        T#5s,         // Step 1: Raw Material Prep
        T#30m,        // Step 2: Polymerization
        T#15m,        // Step 3: Quenching
        T#1h,         // Step 4: Drying
        T#1h30m,      // Step 5: Pelletizing
        T#2h,         // Step 6: Quality Control
        T#3h          // Step 7: Packaging
    ];
END_VAR

METHOD PRIVATE SetTemperatureAndPressure : BOOL
VAR_INPUT
    temp : REAL;
    pressure : REAL;
END_VAR
// Simulated assignment to actual process controllers (replace with hardware IO)
RETURN TRUE;
END_METHOD

// Main Program Logic
timer(IN := TRUE); // Enable the timer unless intentionally disabled

CASE state OF
    0:
        state := 1;
        stepStartTime := TIME();
        timer(IN := FALSE); // Reset timer
    1..7:
        timer(PT := stepDurations[state]);
        IF timer.Q THEN
            state := state + 1;
            stepStartTime := TIME();
            timer(IN := FALSE); // Reset timer before next step
        END_IF;
    8:
        state := 0; // All steps done → Restart
END_CASE;

// Always update setpoints each cycle
UpdateTemperaturesAndPressures();
