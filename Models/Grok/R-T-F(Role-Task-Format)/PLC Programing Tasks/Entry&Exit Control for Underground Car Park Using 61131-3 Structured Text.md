PROGRAM TrafficLightControl
VAR
    (* Inputs *)
    X1 : BOOL; (* Photoelectric switch at ground floor, ON when car passes *)
    X2 : BOOL; (* Photoelectric switch at basement, ON when car passes *)
    M1 : BOOL; (* ON for one scan cycle when car passes from ground to basement *)
    M2 : BOOL; (* ON for one scan cycle when car passes from basement to ground *)
    M3 : BOOL; (* ON for one scan cycle when car exits ground floor *)
    M4 : BOOL; (* ON for one scan cycle when car exits basement *)
    
    (* Outputs *)
    Y1 : BOOL; (* Red lights for both ends, ON = stop *)
    Y2 : BOOL; (* Green lights for both ends, ON = proceed *)
    
    (* Internal variables *)
    M20 : BOOL; (* TRUE if car is in passage from ground floor *)
    M30 : BOOL; (* TRUE if car is in passage from basement *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid sensor state *)
END_VAR

(* Initialize outputs and flags *)
IF NOT (M20 OR M30) THEN
    (* Initial state: passage clear *)
    Y1 := FALSE; (* Red lights OFF *)
    Y2 := TRUE;  (* Green lights ON *)
END_IF;

(* Validate sensor and flag inputs *)
IF (M1 AND M2) OR (M3 AND M4) OR (M1 AND M4) OR (M2 AND M3) THEN
    (* Conflicting directional flags: should not occur *)
    ErrorCode := 1;
    Y1 := TRUE;  (* Red lights ON for safety *)
    Y2 := FALSE; (* Green lights OFF *)
ELSE
    ErrorCode := 0;
END_IF;

(* Track passage occupancy *)
(* Car enters from ground floor *)
IF M1 OR M4 THEN
    M20 := TRUE; (* Set ground-to-basement passage flag *)
END_IF;

(* Car enters from basement *)
IF M2 OR M3 THEN
    M30 := TRUE; (* Set basement-to-ground passage flag *)
END_IF;

(* Car exits from ground floor *)
IF M3 THEN
    M20 := FALSE; (* Clear ground-to-basement passage flag *)
END_IF;

(* Car exits from basement *)
IF M2 THEN
    M30 := FALSE; (* Clear basement-to-ground passage flag *)
END_IF;

(* Light control logic *)
IF M20 OR M30 THEN
    (* Passage occupied: block passage *)
    Y1 := TRUE;  (* Red lights ON *)
    Y2 := FALSE; (* Green lights OFF *)
ELSE
    (* Passage clear: allow passage *)
    Y1 := FALSE; (* Red lights OFF *)
    Y2 := TRUE;  (* Green lights ON *)
END_IF;

END_PROGRAM
