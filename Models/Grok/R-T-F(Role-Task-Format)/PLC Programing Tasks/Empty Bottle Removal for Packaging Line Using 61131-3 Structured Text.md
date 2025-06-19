PROGRAM PackagingLineControl
VAR
    (* Inputs *)
    BottlePresentSensor : BOOL; (* TRUE if a bottle is detected on the conveyor *)
    EmptyBottleSensor : BOOL; (* TRUE if the detected bottle is empty *)
    
    (* Outputs *)
    ConveyorMotor : BOOL := TRUE; (* TRUE to run the conveyor continuously *)
    EjectCylinder : BOOL; (* TRUE to extend cylinder and eject empty bottle *)
    
    (* Internal variables *)
    EjectTimer : TON; (* 500ms timer for ejection duration *)
    ErrorCode : INT := 0; (* 0: Success, 1: Sensor conflict *)
END_VAR

(* Ensure conveyor runs continuously *)
ConveyorMotor := TRUE;

(* Validate sensor inputs *)
IF BottlePresentSensor AND NOT EmptyBottleSensor THEN
    (* Filled bottle detected: no action needed *)
    ErrorCode := 0;
ELSIF BottlePresentSensor AND EmptyBottleSensor THEN
    (* Empty bottle detected: initiate ejection *)
    ErrorCode := 0;
ELSE
    (* No bottle or invalid state *)
    ErrorCode := 0;
END_IF;

(* Ejection logic *)
IF BottlePresentSensor AND EmptyBottleSensor THEN
    (* Empty bottle detected: start ejection timer *)
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE; (* Extend cylinder to eject bottle *)
ELSE
    (* No empty bottle: reset timer if not running *)
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE; (* Retract cylinder *)
    END_IF;
END_IF;

END_PROGRAM
