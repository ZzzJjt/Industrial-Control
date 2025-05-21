(* Program: Feedforward Control for Conveyor Belt Speed *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Adjusts conveyor speed based on predicted material load *)
(* Executes cyclically (e.g., 100 ms) with safety checks *)
PROGRAM PRG_FeedforwardConveyorControl
VAR
    (* Inputs *)
    Predicted_Load : REAL;             (* Predicted material load, kg *)
    EmergencyStop : BOOL;              (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Conveyor_Speed : REAL;            (* Conveyor speed, m/s *)
    LoadError : BOOL;                 (* TRUE if Predicted_Load is invalid *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    Base_Speed : REAL := 1.0;         (* Minimum conveyor speed, m/s *)
    Gain_FF : REAL := 0.02;           (* Feedforward gain, m/s per kg *)
    Speed_Min : REAL := 0.5;          (* Minimum safe speed, m/s *)
    Speed_Max : REAL := 2.0;          (* Maximum safe speed, m/s *)
    Load_Max : REAL := 100.0;         (* Maximum valid load, kg *)
    Load_Min : REAL := 0.0;           (* Minimum valid load, kg *)
    
    (* Internal Variables *)
    ValidLoad : BOOL;                 (* TRUE if Predicted_Load is valid *)
END_VAR

(* Initialize outputs *)
Conveyor_Speed := 0.0;                (* Conveyor stopped *)
LoadError := FALSE;                   (* No initial load error *)
AlarmActive := FALSE;                 (* No initial alarm *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt conveyor and activate alarm *)
    Conveyor_Speed := 0.0;            (* Stop conveyor *)
    LoadError := FALSE;               (* Clear load error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Validate sensor input *)
(* Check for numerical stability and valid range *)
ValidLoad := Predicted_Load >= Load_Min AND Predicted_Load <= Load_Max AND ABS(Predicted_Load) <= 1.0E10;

IF NOT ValidLoad THEN
    (* Invalid load reading: Stop conveyor *)
    Conveyor_Speed := 0.0;            (* Stop conveyor *)
    LoadError := TRUE;                (* Flag load error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
ELSE
    LoadError := FALSE;               (* Clear load error *)
    AlarmActive := FALSE;             (* Clear alarm *)
END_IF;

(* Feedforward control: Calculate conveyor speed *)
(* Formula: Conveyor_Speed = Base_Speed + Gain_FF * Predicted_Load *)
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

(* Clamp conveyor speed within safe operational range *)
IF Conveyor_Speed > Speed_Max THEN
    Conveyor_Speed := Speed_Max;      (* Limit to maximum speed *)
ELSIF Conveyor_Speed < Speed_Min THEN
    Conveyor_Speed := Speed_Min;      (* Limit to minimum speed *)
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* Conveyor stopped if fault or emergency *)
IF EmergencyStop OR LoadError THEN
    Conveyor_Speed := 0.0;            (* Stop conveyor *)
END_IF;

END_PROGRAM
