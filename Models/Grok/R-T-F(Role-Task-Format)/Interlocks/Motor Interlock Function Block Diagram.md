FUNCTION_BLOCK MotorInterlock
(*
    Function Block: MotorInterlock
    Purpose: Implements safe motor start logic by checking the running status of
             associated equipment, allowing motor startup only when all equipment
             is shut down. Reusable for coordinated startup in industrial control systems.
    Standard: IEC 61131-3 Structured Text
    Date: May 20, 2025
*)

(* Input Variables *)
VAR_INPUT
    Equipment1Running : BOOL;  (* TRUE: Equipment 1 is running *)
    Equipment2Running : BOOL;  (* TRUE: Equipment 2 is running *)
    Equipment3Running : BOOL;  (* TRUE: Equipment 3 is running *)
    Execute           : BOOL;   (* TRUE: Enable interlock checks *)
END_VAR

(* Output Variables *)
VAR_OUTPUT
    AllowStart        : BOOL;   (* TRUE: Motor startup allowed *)
    Error             : BOOL;   (* TRUE: Input validation error *)
    ErrorDesc         : STRING[50]; (* Error description *)
    AuditLogEntry     : STRING[80]; (* Audit log message *)
END_VAR

(* Internal Variables *)
VAR
    PrevExecute       : BOOL := FALSE;  (* Previous Execute state *)
END_VAR

(* Internal Methods *)
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
AllowStart := FALSE;
Error := FALSE;
ErrorDesc := '';
AuditLogEntry := '';

(* Check if enabled *)
IF NOT Execute THEN
    LogAuditEntry('MotorInterlock: Disabled');
    RETURN;
END_IF;

(* Validate inputs - optional, assuming digital inputs are reliable *)
(* In practice, add checks for sensor faults if needed *)

(* Interlock logic: Allow start only if all equipment is shut down *)
IF NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running THEN
    AllowStart := TRUE;
    LogAuditEntry('MotorInterlock: All equipment stopped, motor start allowed');
ELSE
    AllowStart := FALSE;
    IF Equipment1Running THEN
        LogAuditEntry('MotorInterlock: Equipment 1 running, motor start blocked');
    ELSIF Equipment2Running THEN
        LogAuditEntry('MotorInterlock: Equipment 2 running, motor start blocked');
    ELSIF Equipment3Running THEN
        LogAuditEntry('MotorInterlock: Equipment 3 running, motor start blocked');
    END_IF;
END_IF;

(* Update previous state *)
PrevExecute := Execute;

END_FUNCTION_BLOCK
