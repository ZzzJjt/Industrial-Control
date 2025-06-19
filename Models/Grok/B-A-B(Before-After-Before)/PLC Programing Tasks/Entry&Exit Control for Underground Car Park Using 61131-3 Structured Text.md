(* Single-Lane Car Park Traffic Control in IEC 61131-3 Structured Text *)
(* Purpose: Prevent collisions by managing vehicle flow with sensors and traffic lights *)

PROGRAM CarParkTrafficControl
VAR
    (* Inputs *)
    X1 : BOOL;                      (* Photoelectric sensor for ground floor entry/exit *)
    X2 : BOOL;                      (* Photoelectric sensor for basement entry/exit *)
    M1 : BOOL;                      (* Scan pulse flag: Ground floor entry detected *)
    M2 : BOOL;                      (* Scan pulse flag: Basement entry detected *)
    M3 : BOOL;                      (* Scan pulse flag: Basement exit detected *)
    M4 : BOOL;                      (* Scan pulse flag: Ground floor exit detected *)

    (* Internal Variables *)
    M20 : BOOL;                     (* TRUE when passage occupied from ground floor *)
    M30 : BOOL;                     (* TRUE when passage occupied from basement *)

    (* Outputs *)
    Y1 : BOOL;                      (* Red lights: ON when passage is occupied *)
    Y2 : BOOL := TRUE;              (* Green lights: ON when passage is free *)
END_VAR

(* Main Control Logic *)
(* 1. Detect Entry into Passage *)
IF M1 OR M4 THEN
    M20 := TRUE;  (* Set ground floor occupancy on entry *)
END_IF;
IF M2 OR M3 THEN
    M30 := TRUE;  (* Set basement occupancy on entry *)
END_IF;

(* 2. Clear Occupancy Flags on Exit *)
IF M3 OR M4 THEN
    M20 := FALSE;  (* Clear ground floor occupancy on exit *)
END_IF;
IF M1 OR M2 THEN
    M30 := FALSE;  (* Clear basement occupancy on exit *)
END_IF;

(* 3. Traffic Light Control *)
IF M20 OR M30 THEN
    Y1 := TRUE;   (* Red lights ON when passage is occupied *)
    Y2 := FALSE;  (* Green lights OFF *)
ELSE
    Y1 := FALSE;  (* Red lights OFF when passage is free *)
    Y2 := TRUE;   (* Green lights ON *)
END_IF;

(* Notes:
   - Initial State: Y2 (Green) = TRUE, Y1 (Red) = FALSE, indicating passage is free
   - Sensors: X1 (ground floor), X2 (basement) detect vehicle presence
   - Scan Flags:
     - M1: Ground entry, M4: Ground exit
     - M2: Basement entry, M3: Basement exit
   - Occupancy:
     - M20: Tracks ground floor vehicle in passage
     - M30: Tracks basement vehicle in passage
   - Safety: Red lights (Y1) ensure no simultaneous entry when passage is occupied
   - Physical Integration:
     - X1, X2: Photoelectric sensors at passage entry/exit points
     - M1-M4: Pulse flags from sensor edge detection (e.g., via PLC input processing)
     - Y1: Relay output for red traffic lights
     - Y2: Relay output for green traffic lights
   - Scalability: Add more sensors or flags for multi-segment passages
   - Maintenance: Add HMI to display passage status (M20, M30) and light states
*)
END_PROGRAM
