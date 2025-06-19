(* Program: SUBSEA_WELLHEAD_INTERLOCKS
   Purpose: Implements emergency interlocks for a subsea gas wellhead.
   Features:
   - Monitors pressure (PT-101), temperature (TT-101), and flow rate (FT-101)
   - Closes master valve (MV-101) and triggers shutdown on overpressure or low flow
   - Shuts down system on overtemperature
   - Prevents automatic restart, requiring manual reset
   - Ensures deterministic control for reliable operation
*)

PROGRAM SUBSEA_WELLHEAD_INTERLOCKS
VAR
    (* Inputs *)
    PT_101 : REAL;              (* Pressure in psi, from PT-101 *)
    TT_101 : REAL;              (* Temperature in °C, from TT-101 *)
    FT_101 : REAL;              (* Flow rate in m³/h, from FT-101 *)
    MANUAL_RESET : BOOL;         (* Manual reset input to clear shutdown *)
    
    (* Outputs *)
    MV_101 : BOOL;              (* Master Valve: TRUE = Open, FALSE = Closed *)
    SHUTDOWN : BOOL;            (* Shutdown state: TRUE = System stopped *)
    ALARM : BOOL;               (* Alarm for operator notification *)
    
    (* Constants *)
    PRESSURE_LIMIT : REAL := 1500.0;    (* Max pressure threshold, psi *)
    TEMP_LIMIT : REAL := 120.0;         (* Max temperature threshold, °C *)
    MIN_FLOW : REAL := 10.0;            (* Min flow rate threshold, m³/h *)
    
    (* Internal Variables *)
    ShutdownLatch : BOOL;               (* Latches shutdown state until reset *)
END_VAR

(* Initialize outputs *)
MV_101 := TRUE;     (* Master valve initially open *)
SHUTDOWN := FALSE;
ALARM := FALSE;
ShutdownLatch := FALSE;

(* Main interlock logic *)
(* Check for emergency conditions *)
IF PT_101 > PRESSURE_LIMIT OR FT_101 < MIN_FLOW THEN
    MV_101 := FALSE;        (* Close master valve *)
    ShutdownLatch := TRUE;  (* Latch shutdown *)
    ALARM := TRUE;          (* Trigger alarm *)
END_IF

IF TT_101 > TEMP_LIMIT THEN
    ShutdownLatch := TRUE;  (* Latch shutdown *)
    ALARM := TRUE;          (* Trigger alarm *)
END_IF

(* Maintain shutdown state *)
IF ShutdownLatch THEN
    MV_101 := FALSE;        (* Ensure master valve remains closed *)
    SHUTDOWN := TRUE;       (* Indicate system shutdown *)
END_IF

(* Manual reset logic *)
IF MANUAL_RESET AND NOT (PT_101 > PRESSURE_LIMIT OR FT_101 < MIN_FLOW OR TT_101 > TEMP_LIMIT) THEN
    ShutdownLatch := FALSE; (* Clear shutdown latch *)
    SHUTDOWN := FALSE;      (* Clear shutdown state *)
    ALARM := FALSE;         (* Clear alarm *)
    MV_101 := TRUE;         (* Reopen master valve *)
END_IF

END_PROGRAM
