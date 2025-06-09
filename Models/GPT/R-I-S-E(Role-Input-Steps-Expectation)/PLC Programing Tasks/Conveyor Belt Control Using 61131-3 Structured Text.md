PROGRAM ConveyorControl
VAR_INPUT
    StationStop1     : BOOL; // Manual stop station 1
    StationStop2     : BOOL; // Manual stop station 2
    StationStop3     : BOOL; // Manual stop station 3
    Sensor1          : BOOL; // Object detection zone 1
    Sensor2          : BOOL; // Object detection zone 2
    Sensor3          : BOOL; // Object detection zone 3
    Sensor4          : BOOL; // Object detection zone 4
    Sensor5          : BOOL; // Object detection zone 5
    AutoMode         : BOOL; // Automatic mode enabled
    ManualMode       : BOOL; // Manual mode enabled
END_VAR

VAR_OUTPUT
    ConveyorRunning  : BOOL; // Conveyor motor control output
END_VAR

// === Safety Priority: Manual Stop Stations ===
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE; // Immediate stop regardless of mode

// === Automatic Mode Logic ===
ELSIF AutoMode THEN
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE; // All sensors detect object → run conveyor
    ELSE
        ConveyorRunning := FALSE; // Any missing object → stop conveyor
    END_IF;

// === Manual Mode Logic ===
ELSIF ManualMode THEN
    ConveyorRunning := TRUE; // Manual mode bypasses sensors

// === Default State ===
ELSE
    ConveyorRunning := FALSE; // No active mode → stop conveyor
END_IF;
