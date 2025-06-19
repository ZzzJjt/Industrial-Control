FUNCTION_BLOCK VESSEL_PRESSURE_RELIEF
(*
    Function Block: VESSEL_PRESSURE_RELIEF
    Purpose: Implements a pressure relief interlock system to prevent vessel overpressure
             by monitoring pressure (PT-101), opening the relief valve (PRV-101) on high
             pressure or faults, latching the open state, and requiring safe conditions
             for closure.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025

    Safety Contribution:
    - Prevents overpressure incidents by opening PRV-101 when pressure exceeds 15 bar,
      avoiding vessel rupture or explosions.
    - Ensures fail-safe operation by opening PRV-101 on sensor or valve faults, protecting
      against unreliable measurements or actuation failures.
    - Latching and controlled reset prevent premature valve closure, ensuring pressure
      remains safe before normal operation resumes.
    - Enhances equipment protection and operational safety by mitigating risks of
      mechanical failure, chemical releases, and personnel injury.
*)

(* Input Variables *)
VAR_INPUT
    PT_101           : REAL;      (* Pressure (bar) from PT-101 *)
    ValveFeedback    : BOOL;      (* TRUE: PRV-101 confirmed open *)
    Execute          : BOOL;      (* TRUE: Enable interlock checks *)
    ManualReset      : BOOL;      (* R_EDGE: Manual reset to allow valve closure *)
    System_Tick_ms   : UDINT;     (* System tick in milliseconds *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    PRV_101_Open     : BOOL;      (* TRUE: Open relief valve PRV-101 *)
    InterlockActive  : BOOL;      (* TRUE: Interlock condition latched *)
    AlarmActive      : BOOL;      (* TRUE: Alarm triggered *)
    AlarmID          : UINT;      (* Alarm code: 0=None, 1=High Pressure,
                                     2=Sensor Fault, 3=Valve Malfunction *)
    Error            : BOOL;      (* TRUE: Input validation error *)
    ErrorDesc        : STRING[50]; (* Error description *)
    AuditLogEntry    : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* Safety thresholds *)
    P_HIGH           : REAL := 15.0;  (* High pressure limit (bar) *)
    P_RESET          : REAL := 12.0;  (* Reset threshold (bar) *)
    
    (* State tracking *)
    PrevExecute      : BOOL := FALSE;  (* Previous Execute state *)
    PrevReset        : BOOL := FALSE;  (* Previous ManualReset state *)
    InterlockLatched : BOOL := FALSE;  (* TRUE: Interlock condition latched *)
    LastAlarmID      : UINT := 0;     (* Last triggered alarm *)
    SensorFault      : BOOL := FALSE;  (* TRUE: Sensor fault detected *)
    ValveFault       : BOOL := FALSE;  (* TRUE: Valve malfunction detected *)
    
    (* Timers *)
    AlarmTimer       : TON;           (* Timer for alarm latching *)
    SensorCheckTimer : TON;           (* Timer for sensor fault detection *)
    LastPressure     : REAL := 0.0;   (* Last valid pressure reading *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE ValidateInputs : BOOL
    (* Validate sensor input *)
    IF PT_101 < -1.0 OR PT_101 > 20.0 THEN
        SensorFault := TRUE;
        Error := TRUE;
        ErrorDesc := 'Invalid pressure reading';
        AuditLogEntry := 'Error: Invalid PT-101 reading';
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

METHOD PRIVATE CheckSensorFault : BOOL
    (* Detect stuck sensor by checking for no change over 5 seconds *)
    IF ABS(PT_101 - LastPressure) < 0.01 THEN
        SensorCheckTimer(IN := TRUE, PT := T#5000ms);
        SensorCheckTimer();
        IF SensorCheckTimer.Q THEN
            RETURN TRUE;
        END_IF;
    ELSE
        SensorCheckTimer(IN := FALSE);
        LastPressure := PT_101;
        RETURN FALSE;
    END_IF;
    RETURN FALSE;
END_METHOD

METHOD PRIVATE LogAuditEntry
    VAR_INPUT
        Message : STRING[80];
    END_VAR
    (* Log audit entry *)
    AuditLogEntry := Message;
    (* In practice, export to HMI or logger: e.g., CALL LogToHMI(AuditLogEntry); *)
END_METHOD

(* Main Execution Logic *)
(* Reset outputs *)
PRV_101_Open := FALSE;
InterlockActive := FALSE;
AlarmActive := FALSE;
AlarmID := 0;
Error := FALSE;
ErrorDesc := '';
AuditLogEntry := '';

(* Check if enabled *)
IF NOT Execute THEN
    IF InterlockLatched THEN
        PRV_101_Open := TRUE;
        InterlockActive := TRUE;
        AlarmActive := TRUE;
        AlarmID := LastAlarmID;
    END_IF;
    RETURN;
END_IF;

(* Validate inputs *)
IF NOT ValidateInputs() THEN
    PRV_101_Open := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 2; (* Sensor Fault *)
    LastAlarmID := 2;
    LogAuditEntry('Interlock: Sensor fault detected, PRV-101 opened');
    RETURN;
END_IF;

(* Check for sensor stuck fault *)
IF CheckSensorFault() THEN
    SensorFault := TRUE;
    PRV_101_Open := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 2; (* Sensor Fault *)
    LastAlarmID := 2;
    LogAuditEntry('Interlock: Stuck sensor detected, PRV-101 opened');
END_IF;

(* Check for valve malfunction *)
IF PRV_101_Open AND NOT ValveFeedback THEN
    ValveFault := TRUE;
    PRV_101_Open := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 3; (* Valve Malfunction *)
    LastAlarmID := 3;
    LogAuditEntry('Interlock: Valve malfunction detected, PRV-101 commanded open');
END_IF;

(* Handle latched interlock *)
IF InterlockLatched THEN
    PRV_101_Open := TRUE;
    InterlockActive := TRUE;
    AlarmActive := TRUE;
    AlarmID := LastAlarmID;
    
    (* Check for manual reset *)
    IF ManualReset AND NOT PrevReset AND PT_101 < P_RESET AND NOT SensorFault AND NOT ValveFault THEN
        InterlockLatched := FALSE;
        AlarmActive := FALSE;
        AlarmID := 0;
        LastAlarmID := 0;
        SensorFault := FALSE;
        ValveFault := FALSE;
        LogAuditEntry('Manual reset: Interlock cleared, PRV-101 allowed to close');
    END_IF;
    PrevReset := ManualReset;
    RETURN;
END_IF;

(* Interlock: High Pressure (> 15 bar) *)
IF PT_101 > P_HIGH THEN
    PRV_101_Open := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 1; (* High Pressure *)
    LastAlarmID := 1;
    LogAuditEntry('Interlock: High pressure detected, PRV-101 opened');
END_IF;

(* Manage alarm latching *)
IF AlarmActive THEN
    AlarmTimer(IN := TRUE, PT := T#10000ms); (* 10s latch *)
    AlarmTimer();
    IF NOT AlarmTimer.Q THEN
        AlarmActive := TRUE;
        AlarmID := LastAlarmID;
    ELSE
        (* Latch persists until manual reset *)
        AlarmActive := InterlockLatched;
        AlarmID := InterlockLatched ? LastAlarmID : 0;
    END_IF;
ELSE
    AlarmTimer(IN := FALSE);
END_IF;

(* Update previous states *)
PrevExecute := Execute;
PrevReset := ManualReset;

END_FUNCTION_BLOCK
