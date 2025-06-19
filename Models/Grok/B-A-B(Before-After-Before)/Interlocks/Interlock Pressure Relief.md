(* Program: VESSEL_PRESSURE_PROTECTION
   Purpose: Implements automatic overpressure protection for an industrial vessel.
   Features:
   - Monitors pressure (PT_101) and opens relief valve (RV_101) on high pressure (>15 bar)
   - Keeps valve open until pressure drops below reset threshold (<12 bar)
   - Fail-safe: Opens valve on sensor or valve faults
   - Logs faults and triggers alarms for operator awareness
   - Uses latching to prevent valve chattering
   - Independent of normal process controls for safety integrity
*)

PROGRAM VESSEL_PRESSURE_PROTECTION
VAR
    (* Inputs *)
    PT_101 : REAL;                  (* Pressure in bar, from pressure sensor *)
    SensorFault : BOOL;             (* TRUE if sensor is faulty, e.g., stuck or offline *)
    ValveFault : BOOL;              (* TRUE if valve fails to respond *)
    ManualReset : BOOL;             (* TRUE for manual reset after fault clearance *)
    AlarmAck : BOOL;                (* TRUE when alarm is acknowledged *)
    
    (* Outputs *)
    RV_101 : BOOL;                  (* Relief Valve: TRUE = Open, FALSE = Closed *)
    ALARM : BOOL;                   (* Alarm for high pressure or faults *)
    FAULT_LOG : BOOL;               (* Logs sensor or valve faults *)
    RELIEF_LATCH : BOOL;            (* Latches relief state until reset *)
    
    (* Constants *)
    HIGH_LIMIT : REAL := 15.0;      (* High pressure threshold, bar *)
    RESET_LIMIT : REAL := 12.0;     (* Reset pressure threshold, bar *)
END_VAR

(* Initialize outputs *)
RV_101 := FALSE;        (* Relief valve initially closed *)
ALARM := FALSE;
FAULT_LOG := FALSE;
RELIEF_LATCH := FALSE;

(* Main interlock logic *)
(* Check for high pressure condition *)
IF PT_101 > HIGH_LIMIT THEN
    RV_101 := TRUE;             (* Open relief valve *)
    RELIEF_LATCH := TRUE;       (* Latch relief state *)
    ALARM := TRUE;              (* Trigger alarm *)
END_IF

(* Fail-safe on sensor or valve fault *)
IF SensorFault OR ValveFault THEN
    RV_101 := TRUE;             (* Open relief valve as fail-safe *)
    RELIEF_LATCH := TRUE;       (* Latch relief state *)
    ALARM := TRUE;              (* Trigger alarm *)
    FAULT_LOG := TRUE;          (* Log fault for maintenance *)
END_IF

(* Maintain relief state *)
IF RELIEF_LATCH THEN
    RV_101 := TRUE;             (* Ensure valve remains open *)
END_IF

(* Reset logic with manual intervention *)
IF RELIEF_LATCH AND ManualReset AND AlarmAck AND PT_101 < RESET_LIMIT AND NOT (SensorFault OR ValveFault) THEN
    RELIEF_LATCH := FALSE;      (* Clear relief latch *)
    RV_101 := FALSE;            (* Close relief valve *)
    ALARM := FALSE;             (* Clear alarm *)
END_IF

(* Clear fault log after maintenance *)
IF FAULT_LOG AND ManualReset AND AlarmAck AND NOT (SensorFault OR ValveFault) THEN
    FAULT_LOG := FALSE;         (* Clear fault log *)
END_IF

END_PROGRAM
