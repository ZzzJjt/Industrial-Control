// Conveyor Belt Control Program (IEC 61131-3 Structured Text)

VAR
    // Input Signals
    StationStop1    : BOOL; // Manual stop station 1
    StationStop2    : BOOL; // Manual stop station 2
    StationStop3    : BOOL; // Manual stop station 3

    Sensor1         : BOOL; // Object detection sensor 1
    Sensor2         : BOOL; // Object detection sensor 2
    Sensor3         : BOOL; // Object detection sensor 3
    Sensor4         : BOOL; // Object detection sensor 4
    Sensor5         : BOOL; // Object detection sensor 5

    AutoMode        : BOOL; // Automatic mode selected
    ManualMode      : BOOL; // Manual mode selected

    // Output Signals
    ConveyorRunning : BOOL; // Controls motor ON/OFF
    ConveyorSpeed   : REAL := 2.0; // Logical constant speed
END_VAR


// --- Safety Priority Logic: Emergency Stop Always Wins ---
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    ConveyorRunning := FALSE;

// --- Automatic Mode: Run only if all 5 sensors detect presence ---
ELSIF AutoMode THEN
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE;
    END_IF

// --- Manual Mode: Run unless manually stopped ---
ELSIF ManualMode THEN
    ConveyorRunning := TRUE;

// --- No Valid Mode Selected: Default to safe stop ---
ELSE
    ConveyorRunning := FALSE;
END_IF
