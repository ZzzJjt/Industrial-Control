FUNCTION_BLOCK CarWashControl
VAR_INPUT
    CarPresentSensor     : BOOL; // TRUE if a car is detected in the wash bay
    HumanDetectedSensor  : BOOL; // TRUE if a person is detected in the wash bay
END_VAR

VAR_OUTPUT
    WashActive           : BOOL; // TRUE when wash cycle is active
    AlarmActive          : BOOL; // TRUE when safety alarm is triggered
    SafeToRun            : BOOL; // TRUE if conditions are safe to run
END_VAR

VAR
    CarPresent           : BOOL;
    HumanDetected        : BOOL;
END_VAR

// Update sensor states
CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

// Priority safety interlock — stop everything if human is present
IF HumanDetected THEN
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE;

// Safe to start wash if car present, area clear, and system safe
ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    WashActive := TRUE;
    AlarmActive := FALSE;

// No car = don't wash
ELSIF NOT CarPresent THEN
    WashActive := FALSE;
END_IF;

// Reset logic: once wash is off and area is clear, system becomes safe again
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF;
