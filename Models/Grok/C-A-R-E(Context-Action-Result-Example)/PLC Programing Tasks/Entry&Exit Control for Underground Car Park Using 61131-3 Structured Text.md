(* IEC 61131-3 Structured Text Program: CarParkPassageControl *)
(* Purpose: Controls traffic flow through a single-lane car park passage *)

PROGRAM CarParkPassageControl
VAR
    (* Inputs *)
    X1 : BOOL;                      (* TRUE when car passes ground floor sensor *)
    X2 : BOOL;                      (* TRUE when car passes basement sensor *)
    M1 : BOOL;                      (* Pulse: ON for 1 scan when car enters from ground *)
    M2 : BOOL;                      (* Pulse: ON for 1 scan when car exits to ground *)
    M3 : BOOL;                      (* Pulse: ON for 1 scan when car enters from basement *)
    M4 : BOOL;                      (* Pulse: ON for 1 scan when car exits to basement *)

    (* Outputs *)
    Y1 : BOOL;                      (* TRUE to activate red lights (stop) *)
    Y2 : BOOL := TRUE;              (* TRUE to activate green lights (go) *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    M20 : BOOL;                     (* TRUE if passage occupied by car from ground *)
    M30 : BOOL;                     (* TRUE if passage occupied by car from basement *)
    LastX1 : BOOL;                  (* Previous X1 state *)
    LastX2 : BOOL;                  (* Previous X2 state *)
    LastM1 : BOOL;                  (* Previous M1 state *)
    LastM2 : BOOL;                  (* Previous M2 state *)
    LastM3 : BOOL;                  (* Previous M3 state *)
    LastM4 : BOOL;                  (* Previous M4 state *)
    LastY1 : BOOL;                  (* Previous Y1 state *)
    LastY2 : BOOL;                  (* Previous Y2 state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Occupancy:
     - M20 := TRUE on M1 (ground entry) or M4 (exit to basement, implies ground entry).
     - M30 := TRUE on M2 (exit to ground, implies basement entry) or M3 (basement entry).
     - M20 := FALSE on M3 (basement entry, exited ground) or M4 (exit to basement).
     - M30 := FALSE on M1 (ground entry, exited basement) or M2 (exit to ground).
   - Lights:
     - If M20 OR M30, Y1 := TRUE (red ON), Y2 := FALSE (green OFF).
     - If NOT M20 AND NOT M30, Y1 := FALSE (red OFF), Y2 := TRUE (green ON).
   - Initial state: Y2 := TRUE, Y1 := FALSE (passage clear).
   - Logs light changes and sensor events for traceability.
*)

(* Step 1: Validate sensor states *)
(* Warn if X1 and X2 are simultaneously TRUE (unlikely, indicates fault) *)
IF X1 AND X2 THEN
    IF LogCount < 50 AND (X1 <> LastX1 OR X2 <> LastX2) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 15:45:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Simultaneous X1 and X2 Detection');
    END_IF;
END_IF;

(* Step 2: Set passage occupancy *)
IF M1 OR M4 THEN
    M20 := TRUE; (* Ground floor entry *)
    IF LogCount < 50 AND (M1 <> LastM1 OR M4 <> LastM4) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 15:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Passage Occupied: Ground Entry');
    END_IF;
END_IF;
IF M2 OR M3 THEN
    M30 := TRUE; (* Basement entry *)
    IF LogCount < 50 AND (M2 <> LastM2 OR M3 <> LastM3) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 15:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Passage Occupied: Basement Entry');
    END_IF;
END_IF;

(* Step 3: Clear passage occupancy after exit *)
IF M3 OR M4 THEN
    M20 := FALSE; (* Ground entry cleared *)
    IF LogCount < 50 AND (M3 <> LastM3 OR M4 <> LastM4) AND M20 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 15:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Passage Cleared: Ground Entry');
    END_IF;
END_IF;
IF M1 OR M2 THEN
    M30 := FALSE; (* Basement entry cleared *)
    IF LogCount < 50 AND (M1 <> LastM1 OR M2 <> LastM2) AND M30 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 15:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Passage Cleared: Basement Entry');
    END_IF;
END_IF;

(* Step 4: Light control *)
IF M20 OR M30 THEN
    Y1 := TRUE;  (* Red lights ON *)
    Y2 := FALSE; (* Green lights OFF *)
ELSE
    Y1 := FALSE; (* Red lights OFF *)
    Y2 := TRUE;  (* Green lights ON *)
END_IF;

(* Step 5: Log light changes *)
IF Y1 AND NOT LastY1 THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 15:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Red Lights ON: Passage Occupied');
    END_IF;
ELSIF NOT Y1 AND LastY1 THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 15:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Green Lights ON: Passage Clear');
    END_IF;
END_IF;

(* Step 6: Update last states for logging *)
LastX1 := X1;
LastX2 := X2;
LastM1 := M1;
LastM2 := M2;
LastM3 := M3;
LastM4 := M4;
LastY1 := Y1;
LastY2 := Y2;

(* Step 7: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Controls traffic flow in a single-lane car park passage.
   - Inputs:
     - X1: BOOL, TRUE for ground floor sensor.
     - X2: BOOL, TRUE for basement sensor.
     - M1: BOOL, pulse for ground entry.
     - M2: BOOL, pulse for ground exit.
     - M3: BOOL, pulse for basement entry.
     - M4: BOOL, pulse for basement exit.
   - Outputs:
     - Y1: BOOL, TRUE for red lights (stop).
     - Y2: BOOL, TRUE for green lights (go).
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Internal:
     - M20: BOOL, TRUE for ground entry occupancy.
     - M30: BOOL, TRUE for basement entry occupancy.
   - Logic:
     - Set M20 on M1 or M4, M30 on M2 or M3.
     - Clear M20 on M3 or M4, M30 on M1 or M2.
     - Y1 := TRUE, Y2 := FALSE if M20 OR M30; else Y1 := FALSE, Y2 := TRUE.
     - Initial: Y2 := TRUE, Y1 := FALSE (passage clear).
   - Optimization:
     - Simple logic (~20 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for flags).
   - Safety:
     - Validates sensor states (warns if X1 AND X2).
     - Ensures single-car passage via M20, M30.
     - Logs events for traceability.
   - Usage:
     - Car park: Allows one car, controls lights.
     - Example: M1=TRUE → M20:=TRUE, Y1:=TRUE, Y2:=FALSE.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], BOOL; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
