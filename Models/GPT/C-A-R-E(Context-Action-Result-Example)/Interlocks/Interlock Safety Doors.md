(* === Variable Declarations === *)
VAR
    DOOR_1_CLOSED      : BOOL;  (* TRUE if door 1 is closed *)
    DOOR_2_CLOSED      : BOOL;  (* TRUE if door 2 is closed *)
    START_REQUEST      : BOOL;  (* Operator start command *)
    RESET_REQUEST      : BOOL;  (* Manual reset command *)
    ReactorRunning     : BOOL;  (* TRUE if reactor is active *)
    ALLOW_START        : BOOL;  (* TRUE if safe to start *)
    EMERGENCY_SHUTDOWN : BOOL;  (* TRUE if emergency triggered *)
    SHUTDOWN_LATCH     : BOOL;  (* TRUE if latched due to unsafe condition *)
    
    (* Outputs *)
    HEATER_ON          : BOOL;
    STIRRER_ON         : BOOL;
    PUMP_ON            : BOOL;
END_VAR

(* === Safety Interlock Logic === *)

(* 1. Check if all doors are closed *)
ALLOW_START := DOOR_1_CLOSED AND DOOR_2_CLOSED;

(* 2. Prevent startup if any door is open *)
IF START_REQUEST AND ALLOW_START AND NOT SHUTDOWN_LATCH THEN
    ReactorRunning := TRUE;
ELSE
    IF NOT ReactorRunning THEN
        ReactorRunning := FALSE;
    END_IF
END_IF

(* 3. Trigger emergency shutdown if door opens during operation *)
IF ReactorRunning AND NOT ALLOW_START THEN
    EMERGENCY_SHUTDOWN := TRUE;
    SHUTDOWN_LATCH := TRUE;
    ReactorRunning := FALSE;
END_IF

(* 4. Latch-based shutdown reset logic *)
IF SHUTDOWN_LATCH THEN
    IF RESET_REQUEST AND ALLOW_START THEN
        SHUTDOWN_LATCH := FALSE;
        EMERGENCY_SHUTDOWN := FALSE;
    END_IF
END_IF

(* 5. Hazardous equipment control *)
IF ReactorRunning THEN
    HEATER_ON := TRUE;
    STIRRER_ON := TRUE;
    PUMP_ON := TRUE;
ELSE
    HEATER_ON := FALSE;
    STIRRER_ON := FALSE;
    PUMP_ON := FALSE;
END_IF
