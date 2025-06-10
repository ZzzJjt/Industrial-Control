(* === Variable Declarations === *)
VAR
    PT_101       : REAL;    (* Pressure in psi *)
    TT_101       : REAL;    (* Temperature in °C *)
    FT_101       : REAL;    (* Flow rate in standard m³/h *)

    MV_101       : BOOL;    (* Output to Master Valve: TRUE = Open, FALSE = Closed *)
    SHUTDOWN     : BOOL;    (* Latching emergency shutdown flag *)
    RESET        : BOOL;    (* Manual reset signal *)
    SAFE_FLOW_MIN: REAL := 10.0;    (* Minimum safe flow rate threshold *)
END_VAR

(* === Emergency Shutdown Logic === *)

(* Pressure Interlock: Shutdown if PT_101 > 1500 psi *)
IF PT_101 > 1500.0 THEN
    SHUTDOWN := TRUE;
END_IF;

(* Flow Interlock: Shutdown if flow drops below minimum safe limit *)
IF FT_101 < SAFE_FLOW_MIN THEN
    SHUTDOWN := TRUE;
END_IF;

(* Temperature Interlock: Shutdown if TT_101 > 120°C *)
IF TT_101 > 120.0 THEN
    SHUTDOWN := TRUE;
END_IF;

(* Latching logic: maintain SHUTDOWN until manual RESET *)
IF RESET THEN
    SHUTDOWN := FALSE;
END_IF;

(* Control the Master Valve (MV-101): Close if SHUTDOWN is active *)
IF SHUTDOWN THEN
    MV_101 := FALSE;  (* CLOSE the master valve *)
ELSE
    MV_101 := TRUE;   (* OPEN the master valve for normal operation *)
END_IF;
