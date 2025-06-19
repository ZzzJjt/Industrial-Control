PROGRAM BottleRemovalSystem
VAR_INPUT
    BottlePresentSensor : BOOL; // Detects any bottle on the conveyor
    EmptyBottleSensor : BOOL; // Determines if the detected bottle is empty
END_VAR

VAR_OUTPUT
    ConveyorMotor : BOOL := TRUE; // Controls the conveyor motor
    EjectCylinder : BOOL; // Controls the pneumatic ejection cylinder
END_VAR

VAR
    EjectTimer : TON; // Timer for ejection duration
END_VAR

// Conveyor runs continuously
ConveyorMotor := TRUE;

// Empty bottle detection and ejection logic
IF BottlePresentSensor AND EmptyBottleSensor THEN
    // Start the ejection timer
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    // Stop the ejection timer
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;

// Additional comments for clarity
// - The conveyor motor runs continuously to transport bottles toward the packaging station.
// - When both BottlePresentSensor and EmptyBottleSensor are TRUE (indicating an empty bottle),
//   the EjectCylinder is activated using a TON timer with a duration of 500 ms.
// - After the timer finishes, the EjectCylinder is deactivated to retract it.
// - This ensures that empty bottles are removed before they reach the packaging zone,
//   while filled bottles pass through the system without interruption.



