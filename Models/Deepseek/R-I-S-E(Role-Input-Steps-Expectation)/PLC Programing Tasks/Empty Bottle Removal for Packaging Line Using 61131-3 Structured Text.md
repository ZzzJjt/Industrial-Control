FUNCTION_BLOCK FB_EmptyBottleEjector
VAR_INPUT
    BottlePresentSensor : BOOL; // TRUE when any bottle is present
    EmptyBottleSensor   : BOOL; // TRUE if bottle is empty
END_VAR

VAR_OUTPUT
    ConveyorMotor   : BOOL := TRUE; // Always running
    EjectCylinder   : BOOL := FALSE;
END_VAR

VAR
    EjectTimer      : TON; // Timer for cylinder extension duration
    EjectActive     : BOOL := FALSE; // Internal flag for ejection state
END_VAR

// Initialize timer
EjectTimer(IN := FALSE, PT := T#500ms);

// --- Main Detection and Ejection Logic ---
IF BottlePresentSensor THEN
    // Check if the bottle is empty
    IF EmptyBottleSensor THEN
        // Start ejection process
        EjectTimer(IN := TRUE); // Start timer for cylinder actuation
        EjectCylinder := TRUE;
        EjectActive := TRUE;
    ELSE
        // Bottle is full — do not eject
        EjectTimer(IN := FALSE);
        IF NOT EjectTimer.Q THEN
            EjectCylinder := FALSE;
        END_IF;
    END_IF;

ELSE
    // No bottle present — reset timer and cylinder
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;

PROGRAM PLC_PRG
VAR
    BottleDetector : FB_EmptyBottleEjector;

    // Simulated inputs
    BottleDetected : BOOL := TRUE;
    EmptyBottle : BOOL := TRUE;

    // Outputs
    MotorOn : BOOL;
    CylinderOut : BOOL;
END_VAR

// Call the function block
BottleDetector(
    BottlePresentSensor := BottleDetected,
    EmptyBottleSensor := EmptyBottle
);

// Map outputs
MotorOn := BottleDetector.ConveyorMotor;
CylinderOut := BottleDetector.EjectCylinder;
