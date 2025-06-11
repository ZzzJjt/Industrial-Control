FUNCTION_BLOCK FB_CarWashController
VAR_INPUT
    CarPresentSensor : BOOL; // TRUE if a car is in the wash bay
    HumanDetectedSensor : BOOL; // TRUE if a person is detected
END_VAR

VAR_OUTPUT
    WashActive : BOOL := FALSE; // Controls the wash process
    AlarmActive : BOOL := FALSE; // Activates alarm if unsafe condition
    SafeToRun : BOOL := TRUE; // Lock flag to allow/disallow wash start
END_VAR

VAR
    // Internal state tracking
    WasHumanDetected : BOOL := FALSE;
END_VAR

// Safety priority: Always check for human presence first
IF HumanDetectedSensor THEN
    // Immediate stop and alarm activation
    WashActive := FALSE;
    AlarmActive := TRUE;
    WasHumanDetected := TRUE;

    // Lock system from starting again until manual reset or safe clearance
    SafeToRun := FALSE;
END_IF;

// If no human was ever detected, check for normal operation
ELSIF NOT WasHumanDetected THEN
    // Allow washing only when a car is present and no human is detected
    IF CarPresentSensor THEN
        WashActive := TRUE;
        AlarmActive := FALSE;
    ELSE
        // Car left or not yet detected — stop washing
        WashActive := FALSE;
        AlarmActive := FALSE;
    END_IF;

// Handle system reset after safety breach
ELSE
    // After a human detection event, wait until both sensors are clear
    IF NOT CarPresentSensor AND NOT HumanDetectedSensor THEN
        // Clear internal lock so a new cycle can begin
        WasHumanDetected := FALSE;
        SafeToRun := TRUE;
        AlarmActive := FALSE;
    END_IF;
END_IF;

PROGRAM PLC_PRG
VAR
    CarWashCtrl : FB_CarWashController;

    // Simulated sensor inputs
    CarPresent : BOOL := FALSE;
    PersonDetected : BOOL := FALSE;

    // Outputs
    WashOn : BOOL := FALSE;
    AlarmOn : BOOL := FALSE;
    CanStart : BOOL := FALSE;
END_VAR

// Call the controller function block
CarWashCtrl(
    CarPresentSensor := CarPresent,
    HumanDetectedSensor := PersonDetected
);

// Map outputs
WashOn := CarWashCtrl.WashActive;
AlarmOn := CarWashCtrl.AlarmActive;
CanStart := CarWashCtrl.SafeToRun;
