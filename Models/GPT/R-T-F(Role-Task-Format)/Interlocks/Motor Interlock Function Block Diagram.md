FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL;
    Equipment2Running : BOOL;
    Equipment3Running : BOOL;
END_VAR
VAR_OUTPUT
    AllowStart : BOOL;
END_VAR

(* Logic: Allow motor start only if all associated equipment is not running *)
AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;

END_FUNCTION_BLOCK
