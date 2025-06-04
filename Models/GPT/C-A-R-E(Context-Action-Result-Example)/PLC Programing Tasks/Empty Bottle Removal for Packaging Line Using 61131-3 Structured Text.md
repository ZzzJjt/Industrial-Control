PROGRAM BottleEjectionControl
VAR
    // Sensor Inputs
    BottlePresentSensor : BOOL;         // TRUE when a bottle is detected
    EmptyBottleSensor   : BOOL;         // TRUE if bottle is empty

    // Actuator Outputs
    ConveyorMotor       : BOOL := TRUE; // Conveyor always runs
    EjectCylinder       : BOOL := FALSE;

    // Internal Timer for 0.5s pulse
    EjectTimer          : TON;
END_VAR

// === Eject Logic for Empty Bottles ===
IF BottlePresentSensor AND EmptyBottleSensor THEN
    // Start timer to activate eject cylinder for 0.5s
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    // Stop timer if no bottle or not empty
    EjectTimer(IN := FALSE);

    // Retract ejector once timer has elapsed
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
