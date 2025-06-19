FUNCTION_BLOCK CarWashSafetyController
VAR_INPUT
    CarPresentSensor     : BOOL;   // TRUE when car is in wash bay
    HumanDetectedSensor  : BOOL;   // TRUE when human is detected
END_VAR

VAR_OUTPUT
    WashActive           : BOOL;   // TRUE when wash is running
    AlarmActive          : BOOL;   // TRUE when human detected during wash
    SafeToRun            : BOOL;   // Internal flag for safety lockout
END_VAR

VAR
    PreviouslyActive     : BOOL := FALSE; // Latch to track wash cycle status
END_VAR

// --- Human Safety Interlock: highest priority ---
IF HumanDetectedSensor THEN
    WashActive := FALSE;              // Immediately stop wash
    AlarmActive := TRUE;             // Trigger alarm
    SafeToRun := FALSE;              // Disable future wash until reset
END_IF

// --- Allow Wash if Safe Conditions Exist ---
IF CarPresentSensor AND NOT HumanDetectedSensor AND SafeToRun THEN
    WashActive := TRUE;              // Start wash cycle
    AlarmActive := FALSE;            // No human detected, alarm off
    PreviouslyActive := TRUE;        // Track that wash has run
END_IF

// --- Reset Logic After Area is Clear and Wash is Idle ---
IF NOT HumanDetectedSensor AND NOT WashActive AND PreviouslyActive THEN
    SafeToRun := TRUE;               // Clear lockout after human gone
    AlarmActive := FALSE;            // Reset alarm
    PreviouslyActive := FALSE;       // Clear latch
END_IF

// --- Safety Fallback: Ensure Wash stays off if no car ---
IF NOT CarPresentSensor THEN
    WashActive := FALSE;             // Prevent running without car
END_IF
