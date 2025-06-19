VAR
    (* Inputs *)
    DOOR_1_CLOSED : BOOL;
    DOOR_2_CLOSED : BOOL;
    DOOR_3_CLOSED : BOOL;
    REACTOR_RUNNING : BOOL;        (* TRUE when reactor is in operation *)
    RESET_REQUEST : BOOL;          (* Manual reset command *)

    (* Outputs *)
    ALLOW_START : BOOL := FALSE;   (* Reactor can be started *)
    EMERGENCY_SHUTDOWN : BOOL := FALSE;  (* Latching shutdown signal *)

    (* Internal *)
    ALL_DOORS_CLOSED : BOOL;
    DOOR_FAULT : BOOL;             (* Sensor fault flag *)
END_VAR

(* Check if all doors are closed *)
ALL_DOORS_CLOSED := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;

(* Fail-safe: Treat undefined (e.g. stuck) sensor signal as door open *)
DOOR_FAULT := NOT (IS_VALID(DOOR_1_CLOSED) AND IS_VALID(DOOR_2_CLOSED) AND IS_VALID(DOOR_3_CLOSED));

(* Allow startup only if all doors are confirmed closed and no prior shutdown *)
IF ALL_DOORS_CLOSED AND NOT EMERGENCY_SHUTDOWN AND NOT DOOR_FAULT THEN
    ALLOW_START := TRUE;
ELSE
    ALLOW_START := FALSE;
END_IF;

(* Emergency shutdown conditions *)
IF (REACTOR_RUNNING AND NOT ALL_DOORS_CLOSED) OR DOOR_FAULT THEN
    EMERGENCY_SHUTDOWN := TRUE;
END_IF;

(* Manual reset logic after all doors are closed again *)
IF RESET_REQUEST AND ALL_DOORS_CLOSED AND NOT DOOR_FAULT THEN
    EMERGENCY_SHUTDOWN := FALSE;
END_IF;

(* Optional: add commands to stop heating, mixing, etc. if shutdown occurs *)
