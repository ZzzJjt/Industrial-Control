```
PROGRAM EmptyBottleRemoval
VAR
    AnyBottleDetected : BOOL := FALSE; // Proximity sensor for any bottle
    EmptyBottleDetected : BOOL := FALSE; // Proximity sensor for empty bottle
    CylinderActivated : BOOL := FALSE; // Pneumatic cylinder activation status
    ConveyorRunning : BOOL := TRUE; // Conveyor operation status
    CylinderActivationDelay : TIME := T#0s; // Delay timer for cylinder activation
    CylinderActivationTime : TIME := T#1s; // Time delay before activating the cylinder
    BottleRemovalDelay : TIME := T#2s; // Delay timer for bottle removal
    BottleRemovalTime : TIME := T#2s; // Time delay before deactivating the cylinder
END_VAR

// Main control logic
IF AnyBottleDetected AND EmptyBottleDetected THEN
    // If an empty bottle is detected, activate the cylinder after a delay
    CylinderActivationDelay := CylinderActivationDelay + T#100ms;
    IF CylinderActivationDelay >= CylinderActivationTime THEN
        CylinderActivated := TRUE;
        CylinderActivationDelay := T#0s;
    END_IF;
ELSIF CylinderActivated THEN
    // After removing the empty bottle, deactivate the cylinder
    BottleRemovalDelay := BottleRemovalDelay + T#100ms;
    IF BottleRemovalDelay >= BottleRemovalTime THEN
        CylinderActivated := FALSE;
        BottleRemovalDelay := T#0s;
    END_IF;
END_IF;

// Conveyor control
IF NOT CylinderActivated THEN
    // Keep the conveyor running unless the cylinder is activated
    ConveyorRunning := TRUE;
ELSE
    // Stop the conveyor when the cylinder is activated
    ConveyorRunning := FALSE;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("Any Bottle Detected", AnyBottleDetected);
// Example: Write("Empty Bottle Detected", EmptyBottleDetected);
// Example: Write("Cylinder Activated", CylinderActivated);
// Example: Write("Conveyor Running", ConveyorRunning);
// Example: Write("Cylinder Activation Delay", CylinderActivationDelay);
// Example: Write("Bottle Removal Delay", BottleRemovalDelay);

END_PROGRAM
```
