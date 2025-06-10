PROGRAM BottleEjectionControl
VAR
    // Sensor Inputs
    BottlePresentSensor : BOOL;       // TRUE when a bottle is detected
    EmptyBottleSensor   : BOOL;       // TRUE when the detected bottle is empty

    // Actuators
    ConveyorMotor : BOOL := TRUE;     // Conveyor runs continuously
    EjectCylinder : BOOL := FALSE;    // Controls the ejector cylinder

    // Timer for ejector activation
    EjectTimer : TON;                 // 500ms pulse timer
END_VAR

// --- Conveyor is always ON ---
ConveyorMotor := TRUE;

// --- Bottle Ejection Logic ---
IF BottlePresentSensor AND EmptyBottleSensor THEN
    // Start ejector timer when empty bottle is detected
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    // Stop timer if condition is not met
    EjectTimer(IN := FALSE);
    
    // Deactivate ejector only after timer expires
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
