PROGRAM PolyethyleneBatchControl
VAR
    // State definitions
    state: INT := 0;
    stepStartTime: TIME;

    // Timer instance
    timer: TON;

    // Process parameters
    rawMatPrepTemp: REAL := 70.0;
    rawMatPrepPressure: REAL := 1.0;
    polymerizationTemp: REAL := 150.0;
    polymerizationPressure: REAL := 30.0;
    quenchingTemp: REAL := 25.0;
    quenchingPressure: REAL := 5.0;
    dryingTemp: REAL := 80.0;
    pelletizingTemp: REAL := 150.0;
    qualityControlTemp: REAL := 25.0;
    packagingStorageTemp: REAL := 20.0;
END_VAR

// --- Helper Methods ---

/// @brief Simulates setting temperature and pressure (replace with real logic)
METHOD PRIVATE SetTemperatureAndPressure : BOOL
VAR_INPUT
    temp: REAL;
    pressure: REAL;
END_VAR
    // Placeholder for actual control logic
    RETURN TRUE;
END_METHOD

/// @brief Updates process conditions based on current state
METHOD PRIVATE UpdateProcessConditions : BOOL
    CASE state OF
        1: SetTemperatureAndPressure(rawMatPrepTemp, rawMatPrepPressure);
        2: SetTemperatureAndPressure(polymerizationTemp, polymerizationPressure);
        3: SetTemperatureAndPressure(quenchingTemp, quenchingPressure);
        4: SetTemperatureAndPressure(dryingTemp, quenchingPressure);
        5: SetTemperatureAndPressure(pelletizingTemp, quenchingPressure);
        6: SetTemperatureAndPressure(qualityControlTemp, quenchingPressure);
        7: SetTemperatureAndPressure(packagingStorageTemp, quenchingPressure);
    END_CASE;
    RETURN TRUE;
END_METHOD

/// @brief Defines time duration for each state
METHOD PRIVATE GetStepDuration : TIME
VAR_INPUT
    step: INT;
END_VAR
    CASE step OF
        1: GetStepDuration := T#5s;
        2: GetStepDuration := T#30m;
        3: GetStepDuration := T#15m;
        4: GetStepDuration := T#1h;
        5: GetStepDuration := T#1h30m;
        6: GetStepDuration := T#2h;
        7: GetStepDuration := T#3h;
        ELSE GetStepDuration := T#0s;
    END_CASE;
END_METHOD

// --- Main Execution Logic ---

// Reset timer at every cycle to avoid unintended behavior
timer(IN := NOT timer.Q);

// Only update conditions if entering a new state
IF state <> prev_state THEN
    UpdateProcessConditions();
    prev_state := state;
END_IF

// Set timer duration based on current state
timer(PT := GetStepDuration(state));

// State transition logic
CASE state OF
    0:
        // Initialization
        state := 1;
        stepStartTime := TIME();

    1:
        IF timer.Q THEN
            state := 2;
            stepStartTime := TIME();
        END_IF;

    2:
        IF timer.Q THEN
            state := 3;
            stepStartTime := TIME();
        END_IF;

    3:
        IF timer.Q THEN
            state := 4;
            stepStartTime := TIME();
        END_IF;

    4:
        IF timer.Q THEN
            state := 5;
            stepStartTime := TIME();
        END_IF;

    5:
        IF timer.Q THEN
            state := 6;
            stepStartTime := TIME();
        END_IF;

    6:
        IF timer.Q THEN
            state := 7;
            stepStartTime := TIME();
        END_IF;

    7:
        IF timer.Q THEN
            state := 0; // Loop back to start or trigger end condition
        END_IF;
END_CASE;

TYPE E_BATCH_STEP :
(
    STEP_INIT := 0,
    STEP_RAW_MATERIAL_PREP,
    STEP_POLYMERIZATION,
    STEP_QUENCHING,
    STEP_DRYING,
    STEP_PELLETIZING,
    STEP_QUALITY_CONTROL,
    STEP_PACKAGING
);
END_TYPE

VAR
    state: E_BATCH_STEP := STEP_INIT;
END_VAR

statusMessage: STRING[80];
statusMessage := 'Step ' + INT_TO_STRING(state) + ' in progress';
