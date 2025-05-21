(* Program: Traffic Light Control for Single-Lane Underground Car Park *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Coordinates entry/exit to prevent simultaneous passage using sensors and lights *)
PROGRAM PRG_TrafficLightControl
VAR
    (* Inputs *)
    X1 : BOOL;                        (* TRUE if car detected at ground floor passage *)
    X2 : BOOL;                        (* TRUE if car detected at basement passage *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    Y1 : BOOL;                        (* Red light: TRUE = ON (stop traffic) *)
    Y2 : BOOL;                        (* Green light: TRUE = ON (allow traffic) *)
    AlarmActive : BOOL;               (* TRUE if emergency stop or error *)
    
    (* Internal Variables *)
    M1 : BOOL;                        (* One-scan flag: Ground floor entry detected *)
    M2 : BOOL;                        (* One-scan flag: Basement exit detected *)
    M3 : BOOL;                        (* One-scan flag: Basement entry detected *)
    M4 : BOOL;                        (* One-scan flag: Ground floor exit detected *)
    M20 : BOOL;                       (* TRUE if car in passage from ground floor *)
    M30 : BOOL;                       (* TRUE if car in passage from basement *)
    LastX1 : BOOL;                    (* Previous state of X1 for edge detection *)
    LastX2 : BOOL;                    (* Previous state of X2 for edge detection *)
    SensorError : BOOL;               (* TRUE if sensor conflict detected *)
END_VAR

(* Initialize outputs and state *)
(* On power-up, allow free movement *)
Y1 := FALSE;                          (* Red light OFF *)
Y2 := TRUE;                           (* Green light ON *)
AlarmActive := FALSE;                 (* No initial alarm *)
M20 := FALSE;                         (* No car from ground floor *)
M30 := FALSE;                         (* No car from basement *)
M1 := FALSE;                          (* Clear memory flags *)
M2 := FALSE;
M3 := FALSE;
M4 := FALSE;
SensorError := FALSE;                 (* No initial sensor error *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt all movement and reset to safe state *)
    Y1 := TRUE;                       (* Red light ON: Stop traffic *)
    Y2 := FALSE;                      (* Green light OFF *)
    AlarmActive := TRUE;              (* Activate alarm *)
    M20 := FALSE;                     (* Clear passage flags *)
    M30 := FALSE;
    M1 := FALSE;                      (* Clear memory flags *)
    M2 := FALSE;
    M3 := FALSE;
    M4 := FALSE;
    SensorError := FALSE;             (* Clear sensor error *)
    RETURN;
END_IF;

(* Validate sensor inputs *)
(* X1 and X2 should not be TRUE simultaneously in a single-lane passage *)
IF X1 AND X2 THEN
    SensorError := TRUE;
    AlarmActive := TRUE;
    Y1 := TRUE;                       (* Red light ON: Stop traffic *)
    Y2 := FALSE;                      (* Green light OFF *)
    M20 := FALSE;                     (* Clear passage flags *)
    M30 := FALSE;
    M1 := FALSE;                      (* Clear memory flags *)
    M2 := FALSE;
    M3 := FALSE;
    M4 := FALSE;
    RETURN;
END_IF;

(* Detect one-scan memory flags using edge detection *)
(* M1: Ground floor entry (X1 rising edge, M30 FALSE) *)
(* M4: Ground floor exit (X1 rising edge, M20 TRUE) *)
IF X1 AND NOT LastX1 THEN
    IF NOT M30 THEN
        M1 := TRUE;                   (* Ground floor entry *)
    ELSIF M20 THEN
        M4 := TRUE;                   (* Ground floor exit *)
    END_IF;
ELSE
    M1 := FALSE;
    M4 := FALSE;
END_IF;

(* M3: Basement entry (X2 rising edge, M20 FALSE) *)
(* M2: Basement exit (X2 rising edge, M30 TRUE) *)
IF X2 AND NOT LastX2 THEN
    IF NOT M20 THEN
        M3 := TRUE;                   (* Basement entry *)
    ELSIF M30 THEN
        M2 := TRUE;                   (* Basement exit *)
    END_IF;
ELSE
    M2 := FALSE;
    M3 := FALSE;
END_IF;

(* Update passage occupancy flags *)
(* M20: Set on ground floor entry (M1, M4), reset on exit (M3, M4) *)
IF M1 OR M4 THEN
    M20 := TRUE;                      (* Car from ground floor in passage *)
END_IF;
IF M3 OR M4 THEN
    M20 := FALSE;                     (* Car from ground floor exited *)
END_IF;

(* M30: Set on basement entry (M2, M3), reset on exit (M1, M2) *)
IF M2 OR M3 THEN
    M30 := TRUE;                      (* Car from basement in passage *)
END_IF;
IF M1 OR M2 THEN
    M30 := FALSE;                     (* Car from basement exited *)
END_IF;

(* Update traffic lights *)
(* If passage occupied (M20 or M30), red light ON *)
(* If passage clear, green light ON *)
IF M20 OR M30 THEN
    Y1 := TRUE;                       (* Red light ON: Stop traffic *)
    Y2 := FALSE;                      (* Green light OFF *)
ELSE
    Y1 := FALSE;                      (* Red light OFF *)
    Y2 := TRUE;                       (* Green light ON: Allow traffic *)
END_IF;

(* Update sensor state history *)
LastX1 := X1;
LastX2 := X2;

(* Ensure safe state on power-up or PLC stop *)
(* Red light ON if sensor error or emergency *)
IF EmergencyStop OR SensorError THEN
    Y1 := TRUE;
    Y2 := FALSE;
END_IF;

END_PROGRAM
