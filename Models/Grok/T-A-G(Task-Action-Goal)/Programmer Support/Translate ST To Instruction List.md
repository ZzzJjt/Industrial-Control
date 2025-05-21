(* Program: Polyethylene Batch Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Manages polyethylene production batch process with modular, scan-friendly state machine *)
(* Conforms to IEC 61131-3, uses single TON timer, and modular condition setting *)
PROGRAM PRG_PolyethyleneBatchControl
VAR
    (* Inputs *)
    Temp_PV : REAL;                   (* Current reactor temperature, °C *)
    Pressure_PV : REAL;               (* Current reactor pressure, bar *)
    FeedValveState : BOOL;            (* TRUE if feed valve is open *)
    CoolantValveState : BOOL;         (* TRUE if coolant valve is open *)
    DischargeValveState : BOOL;       (* TRUE if discharge valve is open *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Temp_SP : REAL;                   (* Temperature setpoint, °C *)
    Pressure_SP : REAL;               (* Pressure setpoint, bar *)
    FeedValveCmd : BOOL;              (* TRUE to open feed valve *)
    CoolantValveCmd : BOOL;           (* TRUE to open coolant valve *)
    DischargeValveCmd : BOOL;         (* TRUE to open discharge valve *)
    BatchComplete : BOOL;             (* TRUE when batch is complete *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency *)
    
    (* Internal Variables *)
    Step : UINT;                      (* State: 0=Idle, 1=Init, 2=Load, 3=Heat, 4=React, 5=Cool, 6=Unload *)
    BatchTimer : TON;                 (* Single timer for all steps *)
    ConditionsMet : BOOL;             (* TRUE if temp/pressure within tolerance *)
    
    (* Parameters *)
    Temp_Tol : REAL := 5.0;           (* Temperature tolerance, ±°C *)
    Pressure_Tol : REAL := 2.0;       (* Pressure tolerance, ±bar *)
    Init_Duration : TIME := T#5s;     (* Initialization duration *)
    Load_Duration : TIME := T#10m;    (* Loading duration *)
    Heat_Duration : TIME := T#20m;    (* Heating duration *)
    React_Duration : TIME := T#30m;   (* Reaction duration *)
    Cool_Duration : TIME := T#15m;    (* Cooling duration *)
    Unload_Duration : TIME := T#5m;   (* Unloading duration *)
END_VAR

(* Method to set temperature and pressure setpoints *)
METHOD PRIVATE UpdateTemperaturesAndPressures
    CASE Step OF
        0: (* Idle *)
            Temp_SP := 25.0;          (* Ambient temperature *)
            Pressure_SP := 1.0;       (* Atmospheric pressure *)
        1: (* Init *)
            Temp_SP := 25.0;          (* Maintain ambient *)
            Pressure_SP := 1.0;       (* Maintain atmospheric *)
        2: (* Load *)
            Temp_SP := 50.0;          (* Preheat for loading *)
            Pressure_SP := 10.0;      (* Low pressure for material flow *)
        3: (* Heat *)
            Temp_SP := 180.0;         (* Reaction temperature *)
            Pressure_SP := 50.0;      (* Reaction pressure *)
        4: (* React *)
            Temp_SP := 180.0;         (* Maintain reaction temperature *)
            Pressure_SP := 50.0;      (* Maintain reaction pressure *)
        5: (* Cool *)
            Temp_SP := 40.0;          (* Cool for safe unloading *)
            Pressure_SP := 5.0;       (* Reduce pressure *)
        6: (* Unload *)
            Temp_SP := 40.0;          (* Maintain safe temperature *)
            Pressure_SP := 1.0;       (* Atmospheric for discharge *)
    END_CASE;
END_METHOD

(* Initialize outputs and state *)
FeedValveCmd := FALSE;                (* Feed valve closed *)
CoolantValveCmd := FALSE;             (* Coolant valve closed *)
DischargeValveCmd := FALSE;           (* Discharge valve closed *)
BatchComplete := FALSE;               (* Batch not complete *)
AlarmActive := FALSE;                 (* No initial alarm *)
Step := 0;                            (* Start in Idle *)
BatchTimer(IN := FALSE, PT := T#0s);  (* Initialize timer *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt process and activate alarm *)
    FeedValveCmd := FALSE;
    CoolantValveCmd := FALSE;
    DischargeValveCmd := FALSE;
    BatchComplete := FALSE;
    AlarmActive := TRUE;
    Step := 0;
    BatchTimer(IN := FALSE);
    RETURN;
END_IF;

(* Check temperature and pressure conditions *)
ConditionsMet := (ABS(Temp_PV - Temp_SP) <= Temp_Tol) AND
                (ABS(Pressure_PV - Pressure_SP) <= Pressure_Tol);

(* State machine *)
CASE Step OF
    0: (* Idle *)
        (* Reset outputs and wait for start *)
        FeedValveCmd := FALSE;
        CoolantValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        BatchComplete := FALSE;
        BatchTimer(IN := FALSE);
        IF NOT AlarmActive THEN
            Step := 1;                (* Start initialization *)
        END_IF;

    1: (* Init *)
        (* Initialize system *)
        FeedValveCmd := FALSE;
        CoolantValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        BatchTimer(IN := TRUE, PT := Init_Duration);
        IF BatchTimer.Q THEN
            BatchTimer(IN := FALSE);
            IF ConditionsMet THEN
                Step := 2;            (* Proceed to Load *)
            ELSE
                AlarmActive := TRUE;
                Step := 0;            (* Fault: return to Idle *)
            END_IF;
        END_IF;

    2: (* Load *)
        (* Load ethylene and catalyst *)
        FeedValveCmd := TRUE;
        CoolantValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        BatchTimer(IN := TRUE, PT := Load_Duration);
        IF BatchTimer.Q THEN
            BatchTimer(IN := FALSE);
            IF FeedValveState AND ConditionsMet THEN
                Step := 3;            (* Proceed to Heat *)
            ELSE
                AlarmActive := TRUE;
                Step := 0;            (* Fault: return to Idle *)
            END_IF;
        END_IF;

    3: (* Heat *)
        (* Heat reactor to reaction conditions *)
        FeedValveCmd := FALSE;
        CoolantValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        BatchTimer(IN := TRUE, PT := Heat_Duration);
        IF BatchTimer.Q THEN
            BatchTimer(IN := FALSE);
            IF ConditionsMet THEN
                Step := 4;            (* Proceed to React *)
            ELSE
                AlarmActive := TRUE;
                Step := 0;            (* Fault: return to Idle *)
            END_IF;
        END_IF;

    4: (* React *)
        (* Maintain reaction conditions *)
        FeedValveCmd := FALSE;
        CoolantValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        BatchTimer(IN := TRUE, PT := React_Duration);
        IF BatchTimer.Q THEN
            BatchTimer(IN := FALSE);
            IF ConditionsMet THEN
                Step := 5;            (* Proceed to Cool *)
            ELSE
                AlarmActive := TRUE;
                Step := 0;            (* Fault: return to Idle *)
            END_IF;
        END_IF;

    5: (* Cool *)
        (* Cool reactor for unloading *)
        FeedValveCmd := FALSE;
        CoolantValveCmd := TRUE;
        DischargeValveCmd := FALSE;
        BatchTimer(IN := TRUE, PT := Cool_Duration);
        IF BatchTimer.Q THEN
            BatchTimer(IN := FALSE);
            IF CoolantValveState AND ConditionsMet THEN
                Step := 6;            (* Proceed to Unload *)
            ELSE
                AlarmActive := TRUE;
                Step := 0;            (* Fault: return to Idle *)
            END_IF;
        END_IF;

    6: (* Unload *)
        (* Unload polyethylene product *)
        FeedValveCmd := FALSE;
        CoolantValveCmd := FALSE;
        DischargeValveCmd := TRUE;
        BatchTimer(IN := TRUE, PT := Unload_Duration);
        IF BatchTimer.Q THEN
            BatchTimer(IN := FALSE);
            IF DischargeValveState AND ConditionsMet THEN
                BatchComplete := TRUE;
                Step := 0;            (* Batch complete: return to Idle *)
            ELSE
                AlarmActive := TRUE;
                Step := 0;            (* Fault: return to Idle *)
            END_IF;
        END_IF;
END_CASE;

(* Update setpoints *)
UpdateTemperaturesAndPressures();

(* Safety checks for extreme conditions *)
IF Pressure_PV > 60.0 OR Pressure_PV < 0.0 OR
   Temperature_PV > 200.0 OR Temperature_PV < 0.0 THEN
    AlarmActive := TRUE;
    FeedValveCmd := FALSE;
    CoolantValveCmd := FALSE;
    DischargeValveCmd := FALSE;
    Step := 0;
    BatchTimer(IN := FALSE);
END_IF;

(* Ensure safe state on power-up or PLC stop *)
IF EmergencyStop OR AlarmActive THEN
    FeedValveCmd := FALSE;
    CoolantValveCmd := FALSE;
    DischargeValveCmd := FALSE;
    BatchComplete := FALSE;
END_IF;

END_PROGRAM
