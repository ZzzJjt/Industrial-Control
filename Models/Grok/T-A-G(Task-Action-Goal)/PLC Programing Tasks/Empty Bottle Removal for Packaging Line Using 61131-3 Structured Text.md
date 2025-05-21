(* Program: Bottle Rejection System for Packaging Line *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Detects and ejects empty bottles using sensors and pneumatic cylinder *)
(* Ensures continuous conveyor operation and safety *)
PROGRAM PRG_BottleRejection
VAR
    (* Inputs *)
    BottlePresentSensor : BOOL;        (* TRUE if bottle detected on conveyor *)
    EmptyBottleSensor : BOOL;         (* TRUE if detected bottle is empty *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    ConveyorMotor : BOOL;             (* TRUE to run conveyor motor *)
    EjectCylinder : BOOL;             (* TRUE to activate pneumatic cylinder *)
    AlarmActive : BOOL;               (* TRUE if emergency stop or error *)
    
    (* Internal Variables *)
    EjectTimer : TON;                 (* 500 ms timer for cylinder activation *)
    State : UINT;                     (* State: 0=Idle, 1=Ejecting *)
    SensorError : BOOL;               (* TRUE if sensor conflict detected *)
END_VAR

(* Initialize outputs and state *)
ConveyorMotor := TRUE;                (* Conveyor runs by default *)
EjectCylinder := FALSE;               (* Cylinder retracted initially *)
AlarmActive := FALSE;                 (* No initial alarm *)
State := 0;                           (* Start in Idle *)
EjectTimer(IN := FALSE, PT := T#500ms); (* 500 ms ejection timer *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt all operations and reset to safe state *)
    ConveyorMotor := FALSE;           (* Stop conveyor *)
    EjectCylinder := FALSE;           (* Retract cylinder *)
    EjectTimer(IN := FALSE);          (* Reset timer *)
    AlarmActive := TRUE;              (* Activate alarm *)
    State := 0;                       (* Return to Idle *)
    SensorError := FALSE;             (* Clear sensor error *)
ELSE
    (* Validate sensor inputs *)
    (* EmptyBottleSensor should only be TRUE if BottlePresentSensor is TRUE *)
    IF EmptyBottleSensor AND NOT BottlePresentSensor THEN
        SensorError := TRUE;
        AlarmActive := TRUE;
        ConveyorMotor := FALSE;       (* Stop conveyor on sensor error *)
        EjectCylinder := FALSE;       (* Ensure cylinder retracted *)
        EjectTimer(IN := FALSE);      (* Reset timer *)
        State := 0;                   (* Return to Idle *)
        RETURN;
    END_IF;

    (* Normal operation: Conveyor runs continuously *)
    ConveyorMotor := TRUE;            (* Keep conveyor running *)

    (* State machine for ejection *)
    CASE State OF
        0: (* Idle *)
            (* Check for empty bottle *)
            IF BottlePresentSensor AND EmptyBottleSensor THEN
                (* Empty bottle detected: Start ejection *)
                EjectCylinder := TRUE;    (* Activate cylinder *)
                EjectTimer(IN := TRUE);   (* Start 500 ms timer *)
                State := 1;               (* Move to Ejecting *)
            END_IF;

        1: (* Ejecting *)
            (* Wait for ejection to complete *)
            IF EjectTimer.Q THEN
                (* Timer complete: Retract cylinder *)
                EjectCylinder := FALSE;   (* Retract cylinder *)
                EjectTimer(IN := FALSE);  (* Reset timer *)
                State := 0;               (* Return to Idle *)
            END_IF;
    END_CASE;
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* Conveyor runs only if no emergency or sensor error *)
IF EmergencyStop OR SensorError THEN
    ConveyorMotor := FALSE;
    EjectCylinder := FALSE;
END_IF;

END_PROGRAM
