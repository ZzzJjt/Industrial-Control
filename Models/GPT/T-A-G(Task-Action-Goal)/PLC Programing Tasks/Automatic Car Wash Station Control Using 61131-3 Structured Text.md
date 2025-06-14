FUNCTION_BLOCK FB_CarWashController
VAR_INPUT
    CarPresentSensor   : BOOL;   // TRUE when car is in the bay
    HumanDetectedSensor: BOOL;   // TRUE when a person is in the zone
END_VAR

VAR_OUTPUT
    WashActive         : BOOL;   // Controls car wash process
    AlarmActive        : BOOL;   // Triggers human-in-zone alarm
    SafeToRun          : BOOL;   // Internal flag to allow washing
END_VAR

VAR
    CarPresent         : BOOL;
    HumanDetected      : BOOL;
END_VAR

// --- Main Logic ---

CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

// 1. Human detected: stop wash, raise alarm, block wash cycle
IF HumanDetected THEN
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE;

// 2. Safe to run: car present, no human, and system is safe
ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    WashActive := TRUE;
    AlarmActive := FALSE;

// 3. Default: wash off when car absent or not safe
ELSE
    WashActive := FALSE;
END_IF

// 4. Reset safe state only when no human and wash is not active
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF
