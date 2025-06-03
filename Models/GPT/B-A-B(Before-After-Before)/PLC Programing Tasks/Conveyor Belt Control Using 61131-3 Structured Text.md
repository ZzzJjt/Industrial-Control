// === Input Variables ===
VAR
    Sensor1, Sensor2, Sensor3, Sensor4, Sensor5 : BOOL; // Object detection sensors
    StationStop1, StationStop2, StationStop3   : BOOL; // Manual station stop buttons
    ManualMode, AutoMode                       : BOOL; // Mode selection switches
END_VAR

// === Output Variables ===
VAR
    ConveyorRunning : BOOL := FALSE; // Conveyor motor control signal
    ConveyorSpeed   : REAL := 2.0;   // Fixed speed in m/s
    ModeStatus      : STRING[20];    // For monitoring
END_VAR

// === Control Logic ===

// Manual Stop always takes highest priority
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE;
    ModeStatus := 'STOPPED BY STATION';

ELSIF AutoMode AND NOT ManualMode THEN
    // Auto mode only runs when all 5 sensors detect items
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
        ModeStatus := 'RUNNING: AUTO MODE';
    ELSE
        ConveyorRunning := FALSE;
        ModeStatus := 'WAITING: MISSING ITEM';
    END_IF;

ELSIF ManualMode AND NOT AutoMode THEN
    // In Manual mode, run as long as no manual stop is active
    IF NOT (StationStop1 OR StationStop2 OR StationStop3) THEN
        ConveyorRunning := TRUE;
        ModeStatus := 'RUNNING: MANUAL MODE';
    ELSE
        ConveyorRunning := FALSE;
        ModeStatus := 'MANUAL STOP TRIGGERED';
    END_IF;

ELSE
    // Fallback: no valid mode selected
    ConveyorRunning := FALSE;
    ModeStatus := 'MODE ERROR OR IDLE';
END_IF;
