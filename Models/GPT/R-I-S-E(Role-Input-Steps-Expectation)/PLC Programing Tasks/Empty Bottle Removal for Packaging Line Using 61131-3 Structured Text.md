FUNCTION_BLOCK EmptyBottleEjector
VAR_INPUT
    BottlePresentSensor : BOOL;         // Detects presence of any bottle
    EmptyBottleSensor   : BOOL;         // Detects if the bottle is empty
END_VAR

VAR_OUTPUT
    ConveyorMotor       : BOOL := TRUE; // Conveyor runs continuously
    EjectCylinder       : BOOL := FALSE;// Pneumatic ejection cylinder
END_VAR

VAR
    EjectTimer          : TON;          // Timer instance for 500 ms pulse
END_VAR

// === Continuous Conveyor Operation ===
ConveyorMotor := TRUE;

// === Ejection Trigger Logic ===
IF BottlePresentSensor AND EmptyBottleSensor THEN
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    EjectTimer(IN := FALSE);
    // Retract cylinder only when the timer is done (500 ms elapsed)
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;

// === Execute Timer ===
EjectTimer();
