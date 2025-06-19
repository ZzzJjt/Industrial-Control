PROGRAM ConveyorControl
VAR
    // Mode selection (set externally)
    AutoMode         : BOOL; // TRUE for automatic mode
    ManualMode       : BOOL; // TRUE for manual mode

    // Operator stations — each can stop the conveyor
    StationStop1     : BOOL;
    StationStop2     : BOOL;
    StationStop3     : BOOL;

    // Item detection sensors
    Sensor1          : BOOL;
    Sensor2          : BOOL;
    Sensor3          : BOOL;
    Sensor4          : BOOL;
    Sensor5          : BOOL;

    // Conveyor outputs
    ConveyorRunning  : BOOL := FALSE; // TRUE when conveyor is running
    ConveyorSpeed    : REAL := 2.0;   // Logical constant speed in m/s (not regulated here)
END_VAR

// 🚨 Step 1: Prioritize manual station stop buttons
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE;

// 🤖 Step 2: Automatic mode — only run if all sensors detect objects
ELSIF AutoMode THEN
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE; // Stop if any item is missing
    END_IF;

// 🧑‍🔧 Step 3: Manual mode — always run unless stopped manually (handled above)
ELSIF ManualMode THEN
    ConveyorRunning := TRUE;

// 🟡 Step 4: If no valid mode selected, ensure safety
ELSE
    ConveyorRunning := FALSE;
END_IF;
