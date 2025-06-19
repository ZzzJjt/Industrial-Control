PROGRAM ConveyorControl
VAR
    (* Inputs *)
    StationStop1 : BOOL; (* Manual stop button at station 1 *)
    StationStop2 : BOOL; (* Manual stop button at station 2 *)
    StationStop3 : BOOL; (* Manual stop button at station 3 *)
    Sensor1 : BOOL; (* Object detection sensor 1 *)
    Sensor2 : BOOL; (* Object detection sensor 2 *)
    Sensor3 : BOOL; (* Object detection sensor 3 *)
    Sensor4 : BOOL; (* Object detection sensor 4 *)
    Sensor5 : BOOL; (* Object detection sensor 5 *)
    AutoMode : BOOL; (* TRUE for automatic sensor-based control *)
    ManualMode : BOOL; (* TRUE for manual control *)
    
    (* Outputs *)
    ConveyorRunning : BOOL; (* TRUE if conveyor is operating *)
    ConveyorSpeed : REAL := 2.0; (* Fixed speed: 2 m/s, logic only *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid mode selection *)
    PrevAutoMode : BOOL; (* Tracks previous AutoMode state *)
    PrevManualMode : BOOL; (* Tracks previous ManualMode state *)
END_VAR

(* Initialize outputs *)
ConveyorRunning := FALSE;
ErrorCode := 0;

(* Validate mode selection *)
IF AutoMode AND ManualMode THEN
    (* Invalid: both modes selected *)
    ErrorCode := 1;
    ConveyorRunning := FALSE;
ELSIF NOT AutoMode AND NOT ManualMode THEN
    (* No mode selected: default to stopped *)
    ErrorCode := 0;
    ConveyorRunning := FALSE;
ELSE
    ErrorCode := 0;
END_IF;

(* Safety override: manual stops take precedence *)
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    (* Any stop button pressed: immediately stop conveyor *)
    ConveyorRunning := FALSE;
ELSE
    (* No stop buttons pressed: evaluate mode *)
    IF AutoMode THEN
        (* Automatic mode: run only if all sensors detect objects *)
        IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
            ConveyorRunning := TRUE;
        ELSE
            ConveyorRunning := FALSE;
        END_IF;
    ELSIF ManualMode THEN
        (* Manual mode: run unless stopped *)
        ConveyorRunning := TRUE;
    END_IF;
END_IF;

(* Update previous mode states for edge detection *)
PrevAutoMode := AutoMode;
PrevManualMode := ManualMode;

END_PROGRAM
