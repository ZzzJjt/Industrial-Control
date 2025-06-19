PROGRAM PLC_PRG
TITLE 'Conveyor Belt Control - Dual Mode Operation'

(*
    Description:
    Implements dual-mode conveyor belt control logic with safety interlocks.
    
    Modes:
    - Manual Mode: Conveyor runs until manually stopped
    - Auto Mode: Conveyor runs only when all 5 sensors detect objects
    
    Safety Logic:
    - Any manual stop button stops the conveyor immediately
    - In auto mode, missing object on any sensor stops the conveyor
    
    Assumptions:
    - Only one mode active at a time
    - Speed is fixed at 2 m/s via external means
    - Sensors are normally closed and go FALSE when object missing
*)

VAR
    // Inputs
    Sensor1 : BOOL := TRUE;        // Object detection sensors
    Sensor2 : BOOL := TRUE;
    Sensor3 : BOOL := TRUE;
    Sensor4 : BOOL := TRUE;
    Sensor5 : BOOL := TRUE;

    StationStop1 : BOOL := FALSE;  // Manual stop buttons
    StationStop2 : BOOL := FALSE;
    StationStop3 : BOOL := FALSE;

    ManualMode : BOOL := FALSE;    // Operator-selected mode
    AutoMode : BOOL := FALSE;

    // Outputs
    ConveyorRunning : BOOL := FALSE;
    ConveyorSpeed : REAL := 2.0;   // Fixed speed (m/s) — used for documentation

    // Internal flags
    AllSensorsDetected : BOOL := FALSE;
    ManualStopTriggered : BOOL := FALSE;
END_VAR

// === MAIN LOGIC ===

// --- Step 1: Check for manual stop commands ---
ManualStopTriggered := StationStop1 OR StationStop2 OR StationStop3;

IF ManualStopTriggered THEN
    ConveyorRunning := FALSE;
    
// --- Step 2: Evaluate Automatic Mode Logic ---
ELSIF AutoMode THEN
    // Check if all sensors detect objects
    AllSensorsDetected := Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5;
    
    IF AllSensorsDetected THEN
        ConveyorRunning := TRUE;
    ELSE
        ConveyorRunning := FALSE;
    END_IF;

// --- Step 3: Evaluate Manual Mode Logic ---
ELSIF ManualMode THEN
    ConveyorRunning := TRUE;

// --- Step 4: Default Case (no mode selected) ---
ELSE
    ConveyorRunning := FALSE;
END_IF;

END_PROGRAM
