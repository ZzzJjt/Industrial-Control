// === Input Declarations ===
VAR
    BottlePresentSensor : BOOL;     // TRUE when any bottle is detected
    EmptyBottleSensor : BOOL;       // TRUE when the bottle is empty
END_VAR

// === Output Declarations ===
VAR
    ConveyorMotor : BOOL := TRUE;   // Conveyor always running
    EjectCylinder : BOOL := FALSE;  // Pneumatic ejector
END_VAR

// === Internal Timer ===
VAR
    EjectTimer : TON;               // Timer to control cylinder activation
END_VAR

// === Core Logic ===
// Conveyor runs continuously
ConveyorMotor := TRUE;

// Detect and eject empty bottles
IF BottlePresentSensor AND EmptyBottleSensor THEN
    EjectTimer(IN := TRUE, PT := T#500ms);   // Start 0.5s timer
    EjectCylinder := TRUE;                   // Activate ejection
ELSE
    EjectTimer(IN := FALSE);                 // Stop timer
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;              // Retract cylinder
    END_IF;
END_IF;
