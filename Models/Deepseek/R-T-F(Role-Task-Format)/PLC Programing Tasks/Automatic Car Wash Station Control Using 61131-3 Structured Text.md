FUNCTION_BLOCK CarWashController
VAR_INPUT
    CarPresentSensor : BOOL; // Sensor detects if a car is present in the wash bay
    HumanDetectedSensor : BOOL; // Sensor detects if a human is present in the danger zone
END_VAR

VAR_OUTPUT
    WashActive : BOOL; // TRUE when the wash is running
    AlarmActive : BOOL; // TRUE when a person is detected in a danger zone
END_VAR

VAR
    SafeToRun : BOOL := FALSE; // Internal flag to allow or lock the start of a new wash cycle
END_VAR

METHOD Execute : BOOL
BEGIN
    // Read sensor inputs
    CarPresent := CarPresentSensor;
    HumanDetected := HumanDetectedSensor;

    // Core logic for controlling the wash cycle
    IF HumanDetected THEN
        // Immediate stop of the wash and activation of the alarm if a human is detected
        WashActive := FALSE;
        AlarmActive := TRUE;
        SafeToRun := FALSE;
    ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
        // Start the wash if a car is present, no human is detected, and the system is safe to run
        WashActive := TRUE;
        AlarmActive := FALSE;
    ELSE
        // Stop the wash if conditions are not met
        WashActive := FALSE;
        AlarmActive := FALSE;
    END_IF;

    // Reset safety flag after conditions are met
    IF NOT HumanDetected AND NOT WashActive THEN
        SafeToRun := TRUE;
    ELSE
        SafeToRun := FALSE;
    END_IF;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK
