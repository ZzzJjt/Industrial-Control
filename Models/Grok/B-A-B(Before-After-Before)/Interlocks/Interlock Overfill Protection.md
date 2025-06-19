(* Program: VESSEL_OVERFILL_PROTECTION
   Purpose: Implements automatic overfill protection for a process vessel.
   Features:
   - Monitors liquid level and closes inlet valve on high-level setpoint
   - Maintains valve closure until level drops below reset threshold
   - Fail-safe closure on sensor or valve faults
   - Latching logic to prevent valve chattering
   - Manual override with supervisory control
*)

PROGRAM VESSEL_OVERFILL_PROTECTION
VAR
    (* Inputs *)
    LevelSensor : REAL;              (* Liquid level in %, 0-100 *)
    SensorFault : BOOL;              (* TRUE if sensor is faulty *)
    ValveFault : BOOL;               (* TRUE if valve is faulty *)
    ManualOverride : BOOL;           (* TRUE for manual override *)
    AlarmAck : BOOL;                 (* TRUE when alarm is acknowledged *)
    
    (* Outputs *)
    INLET_VALVE : BOOL;              (* Inlet valve: TRUE = Open, FALSE = Closed *)
    ALARM : BOOL;                    (* High-level or fault alarm *)
    SHUTDOWN_LATCH : BOOL;           (* Latches shutdown state *)
    
    (* Constants *)
    HIGH_SETPOINT : REAL := 90.0;    (* High-level setpoint, % *)
    RESET_THRESHOLD : REAL := 80.0;  (* Safe reset threshold, % *)
END_VAR

(* Initialize outputs *)
INLET_VALVE := TRUE;    (* Inlet valve initially open *)
ALARM := FALSE;
SHUTDOWN_LATCH := FALSE;

(* Main interlock logic *)
(* Check for high-level condition *)
IF LevelSensor > HIGH_SETPOINT THEN
    INLET_VALVE := FALSE;       (* Close inlet valve *)
    SHUTDOWN_LATCH := TRUE;     (* Latch shutdown *)
    ALARM := TRUE;              (* Trigger alarm *)
END_IF

(* Fail-safe on sensor or valve fault *)
IF SensorFault OR ValveFault THEN
    INLET_VALVE := FALSE;       (* Close inlet valve *)
    SHUTDOWN_LATCH := TRUE;     (* Latch shutdown *)
    ALARM := TRUE;              (* Trigger alarm *)
END_IF

(* Maintain shutdown state *)
IF SHUTDOWN_LATCH THEN
    INLET_VALVE := FALSE;       (* Ensure valve remains closed *)
END_IF

(* Reset logic with manual override *)
IF SHUTDOWN_LATCH AND ManualOverride AND AlarmAck AND LevelSensor < RESET_THRESHOLD AND NOT (SensorFault OR ValveFault) THEN
    SHUTDOWN_LATCH := FALSE;    (* Clear shutdown latch *)
    ALARM := FALSE;             (* Clear alarm *)
    INLET_VALVE := TRUE;        (* Reopen inlet valve *)
END_IF

END_PROGRAM
