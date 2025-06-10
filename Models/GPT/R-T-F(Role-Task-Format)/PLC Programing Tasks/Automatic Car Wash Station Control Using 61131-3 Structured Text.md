PROGRAM CarWashSafetyController
VAR_INPUT
    CarPresentSensor      : BOOL; // Car detected inside wash bay
    HumanDetectedSensor   : BOOL; // Human detected in wash zone
END_VAR

VAR_OUTPUT
    WashActive            : BOOL; // TRUE when wash is running
    AlarmActive           : BOOL; // TRUE when human detected in danger zone
END_VAR

VAR
    SafeToRun             : BOOL := TRUE; // Internal interlock flag
END_VAR

// Mirror sensor input to local flags (optional, for clarity)
VAR
    CarPresent            : BOOL;
    HumanDetected         : BOOL;
END_VAR

// ------------------- Logic Start -------------------
CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

// Priority 1: Emergency – human detected, force shutdown
IF HumanDetected THEN
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE; // Lock system until manual/safe clear
// Priority 2: Safe conditions — car present, no human, safe flag set
ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    WashActive := TRUE;
    AlarmActive := FALSE;
// Priority 3: No car or unsafe — stop wash
ELSE
    WashActive := FALSE;
END_IF

// Reset SafeToRun only when conditions are safe and no wash active
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF
