FUNCTION_BLOCK REACTOR_DOOR_INTERLOCK
(*
    Function Block: REACTOR_DOOR_INTERLOCK
    Purpose: Implements a door-based interlock system for a chemical reactor, monitoring
             safety door statuses to block startup, trigger emergency shutdown, stop
             hazardous processes, and latch conditions until manual reset, with fail-safe
             behavior for sensor faults.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025
*)

(* Input Variables *)
VAR_INPUT
    DOOR_1_CLOSED    : BOOL;      (* TRUE: Door 1 closed, FALSE: Open *)
    DOOR_2_CLOSED    : BOOL;      (* TRUE: Door 2 closed, FALSE: Open *)
    DOOR_3_CLOSED    : BOOL;      (* TRUE: Door 3 closed, FALSE: Open *)
    REACTOR_RUNNING  : BOOL;      (* TRUE: Reactor is running *)
    Execute          : BOOL;      (* TRUE: Enable interlock checks *)
    ManualReset      : BOOL;      (* R_EDGE: Manual reset to clear shutdown *)
    System_Tick_ms   : UDINT;     (* System tick in milliseconds *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    ALLOW_START      : BOOL;      (* TRUE: Reactor startup allowed *)
    EMERGENCY_SHUTDOWN : BOOL;    (* TRUE: Initiate emergency shutdown *)
    STOP_HEATING     : BOOL;      (* TRUE: Stop heater *)
    STOP_MIXING      : BOOL;      (* TRUE: Stop agitator *)
    STOP_PRESSURIZATION : BOOL;   (* TRUE: Stop pressurization *)
    AlarmActive      : BOOL;      (* TRUE: Alarm triggered *)
    AlarmID          : UINT;      (* Alarm code: 0=None, 1=Door Open,
                                     2=Door Open During Run, 3=Sensor Fault *)
    Error            : BOOL;      (* TRUE: Sensor validation error *)
    ErrorDesc        : STRING[50]; (* Error description *)
    AuditLogEntry    : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* State tracking *)
    PrevExecute      : BOOL := FALSE;  (* Previous Execute state *)
    PrevReset        : BOOL := FALSE;  (* Previous ManualReset state *)
    ShutdownLatched  : BOOL := FALSE;  (* TRUE: Shutdown condition latched *)
    LastAlarmID      : UINT := 0;     (* Last triggered alarm *)
    SensorFault      : BOOL := FALSE;  (* TRUE: Sensor fault detected *)
    
    (* Timer for alarm persistence *)
    AlarmTimer       : TON;           (* Timer for alarm latching *)
    SensorCheckTimer : TON;           (* Timer for sensor fault detection *)
    DoorStatesStable : BOOL := TRUE;  (* TRUE: Door states consistent *)
    LastDoor1State   : BOOL := FALSE; (* Last DOOR_1_CLOSED state *)
    LastDoor2State   : BOOL := FALSE; (* Last DOOR_2_CLOSED state *)
    LastDoor3State   : BOOL := FALSE; (* Last DOOR_3_CLOSED state *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE ValidateSensors : BOOL
    (* Validate door sensor signals *)
    (* Check for stuck or inconsistent signals *)
    DoorStatesStable := (DOOR_1_CLOSED = LastDoor1State AND
                        DOOR_2_CLOSED = LastDoor2State AND
                        DOOR_3_CLOSED = LastDoor3State);
    
    IF NOT DoorStatesStable THEN
        SensorCheckTimer(IN := TRUE, PT := T#5000ms); (* 5s to detect stuck *)
        SensorCheckTimer();
        LastDoor1State := DOOR_1_CLOSED;
        LastDoor2State := DOOR_2_CLOSED;
        LastDoor3State := DOOR_3_CLOSED;
        RETURN TRUE; (* Allow state change within 5s *)
    ELSIF SensorCheckTimer.Q THEN
        SensorFault := TRUE;
        Error := TRUE;
        ErrorDesc := 'Stuck door sensor detected';
        AuditLogEntry := 'Error: Stuck door sensor detected';
        RETURN FALSE;
    ELSE
        SensorCheckTimer(IN := FALSE);
        RETURN TRUE;
    END_IF;
END_METHOD

METHOD PRIVATE AllDoorsClosed : BOOL
    (* Check if all doors are closed *)
    RETURN DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;
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
ALLOW_START := FALSE;
EMERGENCY_SHUTDOWN := FALSE;
STOP_HEATING := FALSE;
STOP_MIXING := FALSE;
STOP_PRESSURIZATION := FALSE;
AlarmActive := FALSE;
AlarmID := 0;
Error := FALSE;
ErrorDesc := '';
AuditLogEntry := '';

(* Check if enabled *)
IF NOT Execute THEN
    IF ShutdownLatched THEN
        EMERGENCY_SHUTDOWN := TRUE;
        STOP_HEATING := TRUE;
        STOP_MIXING := TRUE;
        STOP_PRESSURIZATION := TRUE;
        AlarmActive := TRUE;
        AlarmID := LastAlarmID;
    END_IF;
    RETURN;
END_IF;

(* Validate sensors *)
IF NOT ValidateSensors() THEN
    EMERGENCY_SHUTDOWN := TRUE;
    STOP_HEATING := TRUE;
    STOP_MIXING := TRUE;
    STOP_PRESSURIZATION := TRUE;
    ShutdownLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 3; (* Sensor Fault *)
    LastAlarmID := 3;
    LogAuditEntry('Interlock: Sensor fault, shutdown initiated');
    RETURN;
END_IF;

(* Handle latched shutdown *)
IF ShutdownLatched THEN
    EMERGENCY_SHUTDOWN := TRUE;
    STOP_HEATING := TRUE;
    STOP_MIXING := TRUE;
    STOP_PRESSURIZATION := TRUE;
    InterlockActive := TRUE;
    AlarmActive := TRUE;
    AlarmID := LastAlarmID;
    
    (* Check for manual reset *)
    IF ManualReset AND NOT PrevReset AND AllDoorsClosed() AND NOT SensorFault THEN
        ShutdownLatched := FALSE;
        AlarmActive := FALSE;
        AlarmID := 0;
        LastAlarmID := 0;
        SensorFault := FALSE;
        LogAuditEntry('Manual reset: Shutdown cleared');
    END_IF;
    PrevReset := ManualReset;
    RETURN;
END_IF;

(* Interlock 1: Block Startup if Any Door Open *)
IF NOT AllDoorsClosed() THEN
    ALLOW_START := FALSE;
    IF NOT REACTOR_RUNNING THEN
        AlarmActive := TRUE;
        AlarmID := 1; (* Door Open *)
        LastAlarmID := 1;
        LogAuditEntry('Interlock: Door open, startup blocked');
    END_IF;
ELSE
    ALLOW_START := TRUE;
END_IF;

(* Interlock 2: Emergency Shutdown if Door Opens During Operation *)
IF REACTOR_RUNNING AND NOT AllDoorsClosed() THEN
    EMERGENCY_SHUTDOWN := TRUE;
    STOP_HEATING := TRUE;
    STOP_MIXING := TRUE;
    STOP_PRESSURIZATION := TRUE;
    ShutdownLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 2; (* Door Open During Run *)
    LastAlarmID := 2;
    LogAuditEntry('Interlock: Door opened during operation, shutdown initiated');
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
        AlarmActive := ShutdownLatched;
        AlarmID := ShutdownLatched ? LastAlarmID : 0;
    END_IF;
ELSE
    AlarmTimer(IN := FALSE);
END_IF;

(* Update previous states *)
PrevExecute := Execute;
PrevReset := ManualReset;

END_FUNCTION_BLOCK
