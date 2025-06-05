PROGRAM PolyethyleneBatchControl
VAR
    state : INT := 1;
    timer : TON;
    timerStart : BOOL := FALSE;

    // Process parameters
    currentTemp : REAL;
    currentPressure : REAL;

    // Step-specific targets
    rawMatPrepTemp : REAL := 70.0;
    rawMatPrepPressure : REAL := 1.5;
    polymerizationTemp : REAL := 180.0;
    polymerizationPressure : REAL := 5.0;
    quenchTemp : REAL := 50.0;
    quenchPressure : REAL := 2.0;
    dryingTemp : REAL := 100.0;
    dryingPressure : REAL := 1.2;
    pelletizingTemp : REAL := 120.0;
    pelletizingPressure : REAL := 2.5;
    qcTemp : REAL := 25.0;
    qcPressure : REAL := 1.0;
    packagingTemp : REAL := 25.0;
    packagingPressure : REAL := 1.0;

    // Timer durations
    tRawMatPrep : TIME := T#5s;
    tPolymerization : TIME := T#30m;
    tQuenching : TIME := T#2m;
    tDrying : TIME := T#10m;
    tPelletizing : TIME := T#3m;
    tQualityCheck : TIME := T#1m;
    tPackaging : TIME := T#2m;

    // Flags
    processComplete : BOOL := FALSE;
END_VAR

// Abstracted method to update setpoints
METHOD SetTemperatureAndPressure
VAR_INPUT
    temp : REAL;
    pressure : REAL;
END_VAR
currentTemp := temp;
currentPressure := pressure;
END_METHOD

// Main state machine logic
CASE state OF
    1: // Raw Material Preparation
        SetTemperatureAndPressure(temp := rawMatPrepTemp, pressure := rawMatPrepPressure);
        timer(IN := TRUE, PT := tRawMatPrep);
        IF timer.Q THEN
            timer(IN := FALSE);
            state := 2;
        END_IF

    2: // Polymerization
        SetTemperatureAndPressure(temp := polymerizationTemp, pressure := polymerizationPressure);
        timer(IN := TRUE, PT := tPolymerization);
        IF timer.Q THEN
            timer(IN := FALSE);
            state := 3;
        END_IF

    3: // Quenching
        SetTemperatureAndPressure(temp := quenchTemp, pressure := quenchPressure);
        timer(IN := TRUE, PT := tQuenching);
        IF timer.Q THEN
            timer(IN := FALSE);
            state := 4;
        END_IF

    4: // Drying
        SetTemperatureAndPressure(temp := dryingTemp, pressure := dryingPressure);
        timer(IN := TRUE, PT := tDrying);
        IF timer.Q THEN
            timer(IN := FALSE);
            state := 5;
        END_IF

    5: // Pelletizing
        SetTemperatureAndPressure(temp := pelletizingTemp, pressure := pelletizingPressure);
        timer(IN := TRUE, PT := tPelletizing);
        IF timer.Q THEN
            timer(IN := FALSE);
            state := 6;
        END_IF

    6: // Quality Control
        SetTemperatureAndPressure(temp := qcTemp, pressure := qcPressure);
        timer(IN := TRUE, PT := tQualityCheck);
        IF timer.Q THEN
            timer(IN := FALSE);
            state := 7;
        END_IF

    7: // Packaging
        SetTemperatureAndPressure(temp := packagingTemp, pressure := packagingPressure);
        timer(IN := TRUE, PT := tPackaging);
        IF timer.Q THEN
            timer(IN := FALSE);
            processComplete := TRUE;
            state := 8;
        END_IF

    8: // Done
        // Final state – do nothing or log completion
        timer(IN := FALSE);
END_CASE
