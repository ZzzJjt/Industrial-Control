PROGRAM ConveyorControl
VAR
    StationStop1, StationStop2, StationStop3 : BOOL; // Manual stop buttons
    Sensor1, Sensor2, Sensor3, Sensor4, Sensor5 : BOOL; // Object detection sensors
    AutoMode, ManualMode : BOOL; // Operation modes
    ConveyorRunning : BOOL; // Output indicating if the conveyor is operating
    ConveyorSpeed : REAL := 2.0; // Conveyor speed in meters per second (logic only)
END_VAR

// Main control logic
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    // Immediate stop if any manual stop button is pressed
    ConveyorRunning := FALSE;
ELSIF AutoMode THEN
    // Automatic mode: Run conveyor only if all sensors detect objects
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE;
    END_IF;
ELSIF ManualMode THEN
    // Manual mode: Run conveyor unless a manual stop is active
    ConveyorRunning := TRUE;
ELSE
    // Default state: Conveyor is stopped
    ConveyorRunning := FALSE;
END_IF;

// Additional safety check (redundant but ensures safety)
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE;
END_IF;
