PROGRAM CarWashControl
VAR_INPUT
    CarPresentSensor : BOOL; // Detects the presence of a vehicle
    HumanDetectedSensor : BOOL; // Detects a person in the wash area
END_VAR

VAR_OUTPUT
    WashActive : BOOL; // Controls wash cycle activation
    AlarmActive : BOOL; // Triggers safety alarm
    SafeToRun : BOOL; // Governs whether the wash cycle can be safely initiated
END_VAR

VAR
    LastHumanDetected : BOOL; // Stores the previous state of HumanDetectedSensor
    LastWashActive : BOOL; // Stores the previous state of WashActive
END_VAR

// Read sensor inputs
CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

// Control Logic
IF HumanDetected THEN
    // If a person is detected, stop the wash process and trigger the alarm
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE;
ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    // Start the wash process if a car is present, no human is detected, and the system is safe to run
    WashActive := TRUE;
    AlarmActive := FALSE;
ELSE
    // Otherwise, do not start the wash process
    WashActive := FALSE;
    AlarmActive := FALSE;
END_IF;

// Allow system to reset once area is clear and wash has stopped
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF;

// Store current states for future comparison
LastHumanDetected := HumanDetected;
LastWashActive := WashActive;

// Inline comments explaining the logic:
// The control logic ensures that the wash process starts only if a car is present, no human is detected, and the system is in a safe-to-run state.
// If a person is detected, the wash process is immediately halted, and an alarm is triggered.
// The system remains in a safe interlocked state until the area is verified to be clear and the wash has stopped.
// Outputs (WashActive, AlarmActive, SafeToRun) are provided for connection to actuators, HMI displays, or safety relays.
// The logic is modular and maintainable, suitable for further expansion (e.g., timed wash stages).



