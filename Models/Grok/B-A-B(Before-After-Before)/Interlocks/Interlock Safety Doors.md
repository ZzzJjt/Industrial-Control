(* Program: REACTOR_DOOR_INTERLOCKS
   Purpose: Implements safety door interlocks for a chemical reactor.
   Features:
   - Monitors safety door statuses (DOOR_1_CLOSED, DOOR_2_CLOSED, DOOR_3_CLOSED)
   - Prevents reactor startup if any door is open
   - Triggers emergency shutdown if any door opens during operation
   - Deactivates hazardous processes (heater, stirrer, pump) during shutdown
   - Requires manual reset with all doors closed to resume operation
   - Includes alarm for operator notification
*)

PROGRAM REACTOR_DOOR_INTERLOCKS
VAR
    (* Inputs *)
    DOOR_1_CLOSED : BOOL;       (* TRUE if door 1 is closed *)
    DOOR_2_CLOSED : BOOL;       (* TRUE if door 2 is closed *)
    DOOR_3_CLOSED : BOOL;       (* TRUE if door 3 is closed *)
    START_REQUEST : BOOL;       (* TRUE when operator requests reactor start *)
    MANUAL_RESET : BOOL;        (* TRUE for manual reset after shutdown *)
    ALARM_ACK : BOOL;           (* TRUE when alarm is acknowledged *)
    
    (* Outputs *)
    REACTOR_RUNNING : BOOL;     (* TRUE when reactor is operational *)
    HEATER_ON : BOOL;           (* TRUE to activate heater *)
    STIRRER_ON : BOOL;          (* TRUE to activate stirrer *)
    PUMP_ON : BOOL;             (* TRUE to activate pump *)
    ALARM : BOOL;               (* TRUE to trigger alarm *)
    EMERGENCY_SHUTDOWN : BOOL;  (* TRUE during emergency shutdown *)
    
    (* Internal Variables *)
    ALLOW_START : BOOL;         (* TRUE if startup is permitted *)
    SHUTDOWN_LATCH : BOOL;      (* Latches shutdown state until reset *)
    ALL_DOORS_CLOSED : BOOL;    (* TRUE if all doors are closed *)
END_VAR

(* Initialize outputs *)
REACTOR_RUNNING := FALSE;
HEATER_ON := FALSE;
STIRRER_ON := FALSE;
PUMP_ON := FALSE;
ALARM := FALSE;
EMERGENCY_SHUTDOWN := FALSE;
ALLOW_START := FALSE;
SHUTDOWN_LATCH := FALSE;

(* Main interlock logic *)
(* Check door statuses *)
ALL_DOORS_CLOSED := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;

(* Startup logic *)
IF START_REQUEST AND ALL_DOORS_CLOSED AND NOT SHUTDOWN_LATCH THEN
    ALLOW_START := TRUE;
ELSE
    ALLOW_START := FALSE;
END_IF

IF ALLOW_START THEN
    REACTOR_RUNNING := TRUE;
    HEATER_ON := TRUE;
    STIRRER_ON := TRUE;
    PUMP_ON := TRUE;
END_IF

(* Runtime interlock logic *)
IF REACTOR_RUNNING AND NOT ALL_DOORS_CLOSED THEN
    EMERGENCY_SHUTDOWN := TRUE;
    SHUTDOWN_LATCH := TRUE;
    ALARM := TRUE;
END_IF

(* Emergency shutdown actions *)
IF EMERGENCY_SHUTDOWN OR SHUTDOWN_LATCH THEN
    REACTOR_RUNNING := FALSE;
    HEATER_ON := FALSE;
    STIRRER_ON := FALSE;
    PUMP_ON := FALSE;
    EMERGENCY_SHUTDOWN := SHUTDOWN_LATCH; (* Maintain shutdown state *)
END_IF

(* Manual reset logic *)
IF SHUTDOWN_LATCH AND MANUAL_RESET AND ALARM_ACK AND ALL_DOORS_CLOSED THEN
    SHUTDOWN_LATCH := FALSE;
    EMERGENCY_SHUTDOWN := FALSE;
    ALARM := FALSE;
END_IF

END_PROGRAM
