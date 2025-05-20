FUNCTION_BLOCK VESSEL_OVERFILL_PROTECT
(*
    Function Block: VESSEL_OVERFILL_PROTECT
    Purpose: Implements interlock logic to prevent vessel overfilling by monitoring
             liquid level, closing the inlet valve on high level or faults, latching
             the condition, and requiring manual reset with valid sensor and safe level.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025

    Safety Contribution:
    - Prevents overfilling, which could cause spills, environmental damage, or pressure
      buildup leading to vessel rupture.
    - Ensures fail-safe operation by closing the inlet valve on sensor or valve faults,
      protecting against equipment failure.
    - Latching and manual reset prevent unsafe restarts, ensuring operator verification.
    - Enhances environmental compliance by minimizing spill risks and supports plant
      safety by reducing hazards like chemical exposure or explosions.
*)

(* Input Variables *)
VAR_INPUT
    LevelSensor      : REAL;      (* Liquid level (%) from LT-101, 0-100% *)
    ValveFeedback    : BOOL;      (* TRUE: Inlet valve confirmed closed *)
    Execute          : BOOL;      (* TRUE: Enable interlock checks *)
    ManualReset      : BOOL;      (* R_EDGE: Manual reset to clear interlock *)
    System_Tick_ms   : UDINT;     (* System tick in milliseconds *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    InletValveClose  : BOOL;      (* TRUE: Close inlet valve FV-101 *)
    InterlockActive  : BOOL;      (* TRUE: Interlock condition latched *)
    AlarmActive      : BOOL;      (* TRUE: Alarm triggered *)
    AlarmID          : UINT;      (* Alarm code: 0=None, 1=High Level,
                                     2=Sensor Fault, 3=Valve Malfunction *)
    Error            : BOOL;      (* TRUE: Input validation error *)
    ErrorDesc        : STRING[50]; (* Error description *)
    AuditLogEntry    : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* Safety thresholds *)
    LEVEL_HIGH       : REAL := 90.0;  (* High level setpoint (%) *)
    LEVEL_RESET      : REAL := 70.0;  (* Reset threshold (%) *)
    
    (* State tracking *)
    PrevExecute      : BOOL := FALSE;  (* Previous Execute state *)
    PrevReset        : BOOL := FALSE;  (* Previous ManualReset state *)
    InterlockLatched : BOOL := FALSE;  (* TRUE: Interlock condition latched *)
    LastAlarmID      : UINT := 0;     (* Last triggered alarm *)
    SensorFault      : BOOL := FALSE;  (* TRUE: Sensor fault detected *)
    ValveFault       : BOOL := FALSE;  (* TRUE: Valve malfunction detected *)
    
    (* Timer for alarm persistence *)
    AlarmTimer       : TON;           (* Timer for alarm latching *)
    SensorCheckTimer : TON;           (* Timer for sensor fault detection *)
    LastLevel        : REAL := 0.0;   (* Last valid level reading *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE ValidateInputs : BOOL
    (* Validate sensor and valve inputs *)
    IF LevelSensor < -5.0 OR LevelSensor > 105.0 THEN
        SensorFault := TRUE;
        Error := TRUE;
        ErrorDesc := 'Invalid level reading';
        AuditLogEntry := 'Error: Invalid LT-101 reading';
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

METHOD PRIVATE CheckSensorFault : BOOL
    (* Detect stuck sensor by checking for no change over 10 seconds *)
    IF ABS(LevelSensor - LastLevel) < 0.1 THEN
        SensorCheckTimer(IN := TRUE, PT := T#10000ms);
        SensorCheckTimer();
        IF SensorCheckTimer.Q THEN
            RETURN TRUE;
        END_IF;
    ELSE
        SensorCheckTimer(IN := FALSE);
        LastLevel := LevelSensor;
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
InletValveClose := FALSE;
InterlockActive := FALSE;
AlarmActive := FALSE;
AlarmID := 0;
Error := FALSE;
ErrorDesc := '';
AuditLogEntry := '';

(* Check if enabled *)
IF NOT Execute THEN
    IF InterlockLatched THEN
        InletValveClose := TRUE;
        InterlockActive := TRUE;
        AlarmActive := TRUE;
        AlarmID := LastAlarmID;
    END_IF;
    RETURN;
END_IF;

(* Validate inputs *)
IF NOT ValidateInputs() THEN
    InletValveClose := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 2; (* Sensor Fault *)
    LastAlarmID := 2;
    LogAuditEntry('Interlock: Sensor fault detected, FV-101 closed');
    RETURN;
END_IF;

(* Check for sensor stuck fault *)
IF CheckSensorFault() THEN
    SensorFault := TRUE;
    InletValveClose := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 2; (* Sensor Fault *)
    LastAlarmID := 2;
    LogAuditEntry('Interlock: Stuck sensor detected, FV-101 closed');
END_IF;

(* Check for valve malfunction *)
IF InletValveClose AND NOT ValveFeedback THEN
    ValveFault := TRUE;
    InletValveClose := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 3; (* Valve Malfunction *)
    LastAlarmID := 3;
    LogAuditEntry('Interlock: Valve malfunction detected, FV-101 commanded closed');
END_IF;

(* Handle latched interlock *)
IF InterlockLatched THEN
    InletValveClose := TRUE;
    InterlockActive := TRUE;
    AlarmActive := TRUE;
    AlarmID := LastAlarmID;
    
    (* Check for manual reset *)
    IF ManualReset AND NOT PrevReset AND LevelSensor < LEVEL_RESET AND NOT SensorFault AND NOT ValveFault THEN
        InterlockLatched := FALSE;
        AlarmActive := FALSE;
        AlarmID := 0;
        LastAlarmID := 0;
        SensorFault := FALSE;
        ValveFault := FALSE;
        LogAuditEntry('Manual reset: Interlock cleared');
    END_IF;
    PrevReset := ManualReset;
    RETURN;
END_IF;

(* Interlock: High Level (> 90%) *)
IF LevelSensor > LEVEL_HIGH THEN
    InletValveClose := TRUE;
    InterlockActive := TRUE;
    InterlockLatched := TRUE;
    AlarmActive := TRUE;
    AlarmID := 1; (* High Level *)
    LastAlarmID := 1;
    LogAuditEntry('Interlock: High level detected, FV-101 closed');
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
