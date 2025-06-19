FUNCTION_BLOCK CarWashSafetyControl
VAR_INPUT
    CarPresentSensor : BOOL;        // TRUE if a car is detected in the wash bay
    HumanDetectedSensor : BOOL;     // TRUE if a person is detected in the wash zone
END_VAR

VAR_OUTPUT
    WashActive : BOOL;              // TRUE if the wash sequence is currently running
    AlarmActive : BOOL;             // TRUE if the alarm is triggered due to human detection
    SafeToRun : BOOL := TRUE;       // Indicates whether it's safe to start the wash again
END_VAR

VAR
    CarPresent : BOOL;
    HumanDetected : BOOL;
END_VAR

// --- Read inputs ---
CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

// --- Priority logic: stop immediately and lock out if a human is detected ---
IF HumanDetected THEN
    WashActive := FALSE;       // Stop the wash cycle
    AlarmActive := TRUE;       // Trigger alarm
    SafeToRun := FALSE;        // Prevent restart until manually or automatically cleared
ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    WashActive := TRUE;        // Start or resume wash cycle
    AlarmActive := FALSE;      // Clear any active alarm
END_IF

// --- Restore safe state only when no human is present and wash is stopped ---
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF
