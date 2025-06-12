PROGRAM PackagingLineControl
VAR
    BottlePresentSensor : BOOL; // Detects the presence of a bottle on the conveyor
    EmptyBottleSensor : BOOL;   // Identifies whether the bottle is empty
    ConveyorMotor : BOOL := TRUE; // Continuously runs the conveyor
    EjectCylinder : BOOL;        // Activates to remove empty bottles
    EjectTimer : TON;            // Timer for ejection duration (500 ms)
END_VAR

// Conveyor always runs
ConveyorMotor := TRUE;

// Bottle removal logic
IF BottlePresentSensor AND EmptyBottleSensor THEN
    // Activate ejector for 0.5 seconds
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    // Deactivate the timer if no empty bottle is detected
    EjectTimer(IN := FALSE);
    // Retract the eject cylinder once the timer completes
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
