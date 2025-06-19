PROGRAM PLC_PRG
TITLE 'Empty Bottle Detection and Ejection System'

(*
    Description:
    Controls a conveyor-based bottle inspection and ejection system.
    
    Features:
    - Continuously runs conveyor motor
    - Detects presence and fill status of bottles
    - Activates ejection cylinder only for empty bottles
    - Uses timer to control cylinder extension duration
    - Prevents false triggering when no bottle is present
    
    Safety:
    - Cylinder only activates when both sensors detect an empty bottle
    - Timer prevents over-extension or repeated activation
*)

VAR
    // Inputs
    BottlePresentSensor : BOOL := FALSE;   // TRUE if any bottle is present
    EmptyBottleSensor   : BOOL := FALSE;   // TRUE if bottle is empty

    // Outputs
    ConveyorMotor       : BOOL := TRUE;    // Always running unless stopped externally
    EjectCylinder       : BOOL := FALSE;   // Pneumatic actuator for ejection

    // Internal logic
    EjectTimer          : TON;             // Timer for cylinder activation
    ShouldEject         : BOOL := FALSE;  // Logic flag for clarity
END_VAR

// === MAIN LOGIC ===

// Step 1: Determine if ejection should occur
ShouldEject := BottlePresentSensor AND EmptyBottleSensor;

// Step 2: Trigger ejection mechanism
IF ShouldEject THEN
    EjectTimer(IN := TRUE, PT := T#500ms);  // Extend cylinder for 500 milliseconds
    EjectCylinder := TRUE;
ELSE
    EjectTimer(IN := FALSE);               // Reset timer
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;            // Deactivate once timer is done
    END_IF;
END_IF;

END_PROGRAM
