(* Function Block: MotorInterlock
   Purpose: Implements a motor interlock to prevent startup when associated equipment is running.
   Features:
   - Monitors three equipment status inputs (Equipment1Running, Equipment2Running, Equipment3Running)
   - Sets AllowStart output to TRUE only when all equipment is stopped
   - Prevents mechanical conflicts and ensures safe motor startup sequencing
   Function Block Diagram Description:
   +-------------------+
   |   MotorInterlock  |
   |-------------------|
   | Equipment1Running |--> [AND] --> AllowStart
   | Equipment2Running |--> [NOT] --|
   | Equipment3Running |--> [NOT] --|
   +-------------------+
   - Inputs are inverted (NOT) to check for stopped state
   - AND logic ensures all equipment must be stopped for AllowStart := TRUE
*)

FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL;    (* TRUE if equipment 1 is running *)
    Equipment2Running : BOOL;    (* TRUE if equipment 2 is running *)
    Equipment3Running : BOOL;    (* TRUE if equipment 3 is running *)
END_VAR
VAR_OUTPUT
    AllowStart : BOOL;           (* TRUE to allow motor startup *)
END_VAR

(* Interlock logic *)
AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;

END_FUNCTION_BLOCK
