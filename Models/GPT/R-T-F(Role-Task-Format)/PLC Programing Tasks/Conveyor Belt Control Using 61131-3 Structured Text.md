PROGRAM ConveyorControl
VAR
    // Inputs
    StationStop1     : BOOL;
    StationStop2     : BOOL;
    StationStop3     : BOOL;
    Sensor1          : BOOL;
    Sensor2          : BOOL;
    Sensor3          : BOOL;
    Sensor4          : BOOL;
    Sensor5          : BOOL;
    AutoMode         : BOOL;
    ManualMode       : BOOL;

    // Output
    ConveyorRunning  : BOOL := FALSE;

    // Constants (logic only, not for motor control)
    ConveyorSpeed    : REAL := 2.0; // meters per second
END_VAR

// --- Priority 1: Manual Emergency Stop Overrides All ---
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE;

// --- Priority 2: Automatic Mode Control ---
ELSIF AutoMode THEN
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE;
    END_IF

// --- Priority 3: Manual Mode Control ---
ELSIF ManualMode THEN
    ConveyorRunning := TRUE;

// --- Default State: Safe Shutdown ---
ELSE
    ConveyorRunning := FALSE;
END_IF;
