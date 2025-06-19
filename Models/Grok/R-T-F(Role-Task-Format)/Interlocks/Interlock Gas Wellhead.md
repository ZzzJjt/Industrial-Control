FUNCTION_BLOCK SUBSEA_WELLHEAD_INTERLOCKS
(*
    Function Block: SUBSEA_WELLHEAD_INTERLOCKS
    Purpose: Implements emergency interlock logic for a subsea gas wellhead, monitoring
             pressure (PT), temperature (TT), and flow rate (FT) to perform shutdown
             actions and latch conditions, requiring manual reset.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025
*)

(* Input Variables *)
VAR_INPUT
    PT_101           : REAL;      (* Pressure (psi) from PT-101 *)
    TT_101           : REAL;      (* Temperature (°C) from TT-101 *)
    FT_101           : REAL;      (* Flow rate (m³/h) from FT-101 *)
    Execute          : BOOL;      (* TRUE: Enable interlock checks *)
    Manual_Reset     : BOOL;      (* R_EDGE: Manual reset to clear shutdown *)
    System_Tick_ms   : UDINT;     (* System tick in milliseconds *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    MV_101_Closed    : BOOL;      (* TRUE: Close master valve MV-101 *)
    Shutdown_Active  : BOOL;      (* TRUE: System shutdown initiated *)
    Alarm_Active     : BOOL;      (* TRUE: Alarm triggered *)
    Alarm_ID         : UINT;      (* Alarm code: 0=None, 1=High Pressure,
                                     2=Low Flow, 3=High Temperature *)
    Error            : BOOL;      (* TRUE: Input validation error *)
    Error_Desc       : STRING[50]; (* Error description *)
    Audit_Log_Entry  : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    (* Safety thresholds *)
    P_MAX            : REAL := 1500.0; (* High pressure limit (psi) *)
    F_MIN            : REAL := 10.0;   (* Low flow threshold (m³/h) *)
    T_MAX            : REAL := 120.0;  (* High temperature limit (°C) *)
    
    (* State tracking *)
    Prev_Execute     : BOOL := FALSE;  (* Previous Execute state *)
    Prev_Reset       : BOOL := FALSE;  (* Previous Manual_Reset state *)
    Shutdown_Latched : BOOL := FALSE;  (* TRUE: Shutdown condition latched *)
    Last_Alarm_ID    : UINT := 0;     (* Last triggered alarm *)
    
    (* Timer for alarm persistence *)
    Alarm_Timer      : TON;           (* Timer for alarm latching *)
END_VAR

(* Internal Methods *)
METHOD PRIVATE ValidateInputs : BOOL
    (* Validate sensor inputs *)
    IF PT_101 < 0.0 OR PT_101 > 2000.0 THEN
        Error := TRUE;
        Error_Desc := 'Invalid pressure reading';
        Audit_Log_Entry := 'Error: Invalid PT-101 reading';
        RETURN FALSE;
    END_IF;
    IF TT_101 < -50.0 OR TT_101 > 200.0 THEN
        Error := TRUE;
        Error_Desc := 'Invalid temperature reading';
        Audit_Log_Entry := 'Error: Invalid TT-101 reading';
        RETURN FALSE;
    END_IF;
    IF FT_101 < 0.0 OR FT_101 > 1000.0 THEN
        Error := TRUE;
        Error_Desc := 'Invalid flow reading';
        Audit_Log_Entry := 'Error: Invalid FT-101 reading';
        RETURN FALSE;
    END_IF;
    RETURN TRUE;
END_METHOD

METHOD PRIVATE LogAuditEntry
    VAR_INPUT
        Message : STRING[80];
    END_VAR
    (* Log audit entry *)
    Audit_Log_Entry := Message;
    (* In practice, export to HMI or logger: e.g., CALL LogToHMI(Audit_Log_Entry); *)
END_METHOD

(* Main Execution Logic *)
(* Reset outputs *)
MV_101_Closed := FALSE;
Shutdown_Active := FALSE;
Alarm_Active := FALSE;
Alarm_ID := 0;
Error := FALSE;
Error_Desc := '';
Audit_Log_Entry := '';

(* Check if enabled *)
IF NOT Execute THEN
    IF Shutdown_Latched THEN
        Shutdown_Active := TRUE;
        MV_101_Closed := TRUE;
        Alarm_Active := TRUE;
        Alarm_ID := Last_Alarm_ID;
    END_IF;
    RETURN;
END_IF;

(* Validate inputs *)
IF NOT ValidateInputs() THEN
    RETURN;
END_IF;

(* Handle latched shutdown *)
IF Shutdown_Latched THEN
    MV_101_Closed := TRUE;
    Shutdown_Active := TRUE;
    Alarm_Active := TRUE;
    Alarm_ID := Last_Alarm_ID;
    
    (* Check for manual reset *)
    IF Manual_Reset AND NOT Prev_Reset THEN
        Shutdown_Latched := FALSE;
        Alarm_Active := FALSE;
        Alarm_ID := 0;
        Last_Alarm_ID := 0;
        LogAuditEntry('Manual reset: Shutdown cleared');
    END_IF;
    Prev_Reset := Manual_Reset;
    RETURN;
END_IF;

(* Interlock 1: High Pressure (> 1500 psi) *)
IF PT_101 > P_MAX THEN
    MV_101_Closed := TRUE;
    Shutdown_Active := TRUE;
    Shutdown_Latched := TRUE;
    Alarm_Active := TRUE;
    Alarm_ID := 1; (* High Pressure *)
    Last_Alarm_ID := 1;
    LogAuditEntry('Interlock 1: High pressure detected, MV-101 closed, shutdown initiated');
END_IF;

(* Interlock 2: Low Flow (< 10 m³/h) *)
IF FT_101 < F_MIN THEN
    MV_101_Closed := TRUE;
    Shutdown_Active := TRUE;
    Shutdown_Latched := TRUE;
    Alarm_Active := TRUE;
    Alarm_ID := 2; (* Low Flow *)
    Last_Alarm_ID := 2;
    LogAuditEntry('Interlock 2: Low flow detected, MV-101 closed, shutdown initiated');
END_IF;

(* Interlock 3: High Temperature (> 120°C) *)
IF TT_101 > T_MAX THEN
    MV_101_Closed := TRUE;
    Shutdown_Active := TRUE;
    Shutdown_Latched := TRUE;
    Alarm_Active := TRUE;
    Alarm_ID := 3; (* High Temperature *)
    Last_Alarm_ID := 3;
    LogAuditEntry('Interlock 3: High temperature detected, MV-101 closed, shutdown initiated');
END_IF;

(* Manage alarm latching *)
IF Alarm_Active THEN
    Alarm_Timer(IN := TRUE, PT := T#10000ms); (* 10s latch *)
    Alarm_Timer();
    IF NOT Alarm_Timer.Q THEN
        Alarm_Active := TRUE;
        Alarm_ID := Last_Alarm_ID;
    ELSE
        (* Latch persists until manual reset *)
        Alarm_Active := Shutdown_Latched;
        Alarm_ID := Shutdown_Latched ? Last_Alarm_ID : 0;
    END_IF;
ELSE
    Alarm_Timer(IN := FALSE);
END_IF;

(* Update previous states *)
Prev_Execute := Execute;
Prev_Reset := Manual_Reset;

END_FUNCTION_BLOCK
