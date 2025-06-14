VAR
    BottlePresentSensor : BOOL; // TRUE if any bottle is detected
    EmptyBottleSensor : BOOL;   // TRUE if the bottle is empty
    ConveyorMotor : BOOL := TRUE; // Conveyor runs continuously
    EjectCylinder : BOOL := FALSE; // Output to control the pneumatic ejection cylinder
    EjectTimer : TON; // Timer instance for 500ms ejection
END_VAR

// Conveyor always runs
ConveyorMotor := TRUE;

// Ejection logic
IF BottlePresentSensor AND EmptyBottleSensor THEN
    // Start the ejection timer and extend the cylinder
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    // Stop the timer if no empty bottle is detected
    EjectTimer(IN := FALSE);
    // Retract the cylinder when the timer finishes
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
