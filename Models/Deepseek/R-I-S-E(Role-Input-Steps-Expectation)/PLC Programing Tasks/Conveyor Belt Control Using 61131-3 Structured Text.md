FUNCTION_BLOCK FB_ConveyorController
VAR_INPUT
    // Operator Station Inputs
    StationStop1 : BOOL; // Manual stop from station 1
    StationStop2 : BOOL; // Manual stop from station 2
    StationStop3 : BOOL; // Manual stop from station 3

    // Object Detection Sensors
    Sensor1 : BOOL;
    Sensor2 : BOOL;
    Sensor3 : BOOL;
    Sensor4 : BOOL;
    Sensor5 : BOOL;

    // Operating Mode Selection
    AutoMode : BOOL;   // TRUE = automatic control
    ManualMode : BOOL; // TRUE = manual override
END_VAR

VAR_OUTPUT
    ConveyorRunning : BOOL := FALSE; // Controls conveyor motor
END_VAR

VAR
    EmergencyStopTriggered : BOOL := FALSE;
END_VAR

// --- STEP 1: Check for emergency stop conditions ---
EmergencyStopTriggered :=
    StationStop1 OR
    StationStop2 OR
    StationStop3;

IF EmergencyStopTriggered THEN
    ConveyorRunning := FALSE;
    
// --- STEP 2: Evaluate operating mode ---
ELSIF AutoMode THEN
    // In auto mode: run only if all sensors detect objects
    IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE;
    END_IF;

ELSIF ManualMode THEN
    // In manual mode: ignore sensors but respect stops
    ConveyorRunning := TRUE;

// --- STEP 3: Default fallback (safety) ---
ELSE
    ConveyorRunning := FALSE;
END_IF;

PROGRAM PLC_PRG
VAR
    ConveyorCtrl : FB_ConveyorController;

    // Simulated inputs
    Stop1 : BOOL := FALSE;
    Stop2 : BOOL := FALSE;
    Stop3 : BOOL := FALSE;

    Sens1 : BOOL := TRUE;
    Sens2 : BOOL := TRUE;
    Sens3 : BOOL := TRUE;
    Sens4 : BOOL := TRUE;
    Sens5 : BOOL := TRUE;

    Auto : BOOL := TRUE;
    Manual : BOOL := FALSE;

    // Output
    MotorOn : BOOL := FALSE;
END_VAR

// Call the controller function block
ConveyorCtrl(
    StationStop1 := Stop1,
    StationStop2 := Stop2,
    StationStop3 := Stop3,
    Sensor1 := Sens1,
    Sensor2 := Sens2,
    Sensor3 := Sens3,
    Sensor4 := Sens4,
    Sensor5 := Sens5,
    AutoMode := Auto,
    ManualMode := Manual
);

// Map output
MotorOn := ConveyorCtrl.ConveyorRunning;
