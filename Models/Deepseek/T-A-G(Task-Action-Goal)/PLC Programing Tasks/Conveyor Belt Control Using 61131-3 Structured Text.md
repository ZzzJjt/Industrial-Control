PROGRAM ConveyorControl
VAR_INPUT
    StationStop1 : BOOL; // Manual stop at station 1
    StationStop2 : BOOL; // Manual stop at station 2
    StationStop3 : BOOL; // Manual stop at station 3
    Sensor1 : BOOL; // Object detection sensor 1
    Sensor2 : BOOL; // Object detection sensor 2
    Sensor3 : BOOL; // Object detection sensor 3
    Sensor4 : BOOL; // Object detection sensor 4
    Sensor5 : BOOL; // Object detection sensor 5
    AutoMode : BOOL; // Automatic mode selection
    ManualMode : BOOL; // Manual mode selection
END_VAR

VAR_OUTPUT
    ConveyorRunning : BOOL; // Controls the conveyor’s running status
    ConveyorSpeed : REAL := 2.0; // Logical representation of constant speed (m/s)
END_VAR

// Main execution loop
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    // Safety override: Stop the conveyor if any manual stop is activated
    ConveyorRunning := FALSE;
ELSEIF AutoMode THEN
    // Automatic mode: Run conveyor only if all sensors detect objects
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE;
    END_IF;
ELSEIF ManualMode THEN
    // Manual mode: Run conveyor unless manually stopped
    ConveyorRunning := TRUE;
ELSE
    // Fallback state: Keep the conveyor stopped if neither mode is active
    ConveyorRunning := FALSE;
END_IF;

// Additional comments for clarity
// - If any station stop button (StationStop1, StationStop2, StationStop3) is pressed,
//   the conveyor is immediately stopped regardless of the current mode.
// - In AutoMode, the conveyor runs only if all five sensors (Sensor1 to Sensor5) detect objects.
// - In ManualMode, the conveyor runs unless any station stop button is pressed.
// - If neither AutoMode nor ManualMode is selected, the conveyor is kept stopped.
// - The conveyor speed is set to a constant value of 2.0 m/s, represented logically.



