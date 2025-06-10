(* === Variable Declarations === *)
VAR
    PT_101           : REAL;      (* Vessel pressure in bar *)
    HighPressureLimit: REAL := 15.0;   (* Relief valve trigger threshold *)
    ResetThreshold   : REAL := 12.0;   (* Reset threshold after pressure drop *)

    RELIEF_VALVE     : BOOL;      (* Output: TRUE = Valve Open, FALSE = Closed *)

    SensorFault      : BOOL;      (* TRUE if pressure sensor is faulty *)
    ValveFault       : BOOL;      (* TRUE if valve feedback indicates failure *)
    InterlockActive  : BOOL;      (* Latching flag to maintain open state *)
    RESET            : BOOL;      (* Manual reset trigger *)
END_VAR

(* === Interlock Activation Logic === *)

(* Trigger interlock if pressure exceeds high limit or faults are detected *)
IF (PT_101 > HighPressureLimit) OR SensorFault OR ValveFault THEN
    InterlockActive := TRUE;
END_IF;

(* Allow reset only if pressure is below reset threshold AND faults cleared *)
IF RESET AND (PT_101 < ResetThreshold) AND NOT SensorFault AND NOT ValveFault THEN
    InterlockActive := FALSE;
END_IF;

(* Valve Control Logic — Fail-safe Open *)
IF InterlockActive THEN
    RELIEF_VALVE := TRUE;  (* OPEN relief valve *)
ELSE
    RELIEF_VALVE := FALSE; (* CLOSE relief valve *)
END_IF;
