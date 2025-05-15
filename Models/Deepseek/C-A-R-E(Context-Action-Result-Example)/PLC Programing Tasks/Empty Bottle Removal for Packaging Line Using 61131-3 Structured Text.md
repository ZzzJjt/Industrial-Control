PROGRAM BottleRemovalControl
VAR_INPUT
    BottlePresentSensor : BOOL; // Detects the presence of a bottle
    EmptyBottleSensor : BOOL;   // Detects whether the bottle is empty
END_VAR

VAR_OUTPUT
    ConveyorMotor : BOOL := TRUE; // Keeps bottles moving toward the packaging station
    EjectCylinder : BOOL;        // Pushes empty bottles off the conveyor
END_VAR

VAR
    EjectTimer : TON;            // Timer for activating the ejector cylinder
END_VAR

// Continuously run the conveyor motor
ConveyorMotor := TRUE;

// Eject logic for empty bottles
IF BottlePresentSensor AND EmptyBottleSensor THEN
    // Activate ejector for 0.5 seconds
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    // Deactivate the ejector after the timer expires
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;

// Inline comments explaining the logic:
// The conveyor motor runs continuously.
// If both sensors detect an empty bottle, activate the ejector for 0.5 seconds.
// Filled bottles continue through the line uninterrupted.
// A timer ensures the cylinder retracts after each ejection, preventing mechanical wear from constant actuation.



