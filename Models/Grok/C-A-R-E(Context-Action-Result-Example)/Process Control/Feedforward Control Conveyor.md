(* IEC 61131-3 Structured Text Program: ConveyorFeedforwardControl *)
(* Purpose: Implements feedforward control for conveyor belt speed *)

PROGRAM ConveyorFeedforwardControl
VAR
    (* Inputs *)
    Predicted_Load : REAL;           (* Predicted load from sensor, kg *)

    (* Outputs *)
    Conveyor_Speed : REAL;          (* Conveyor motor speed, m/s *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Base_Speed : REAL := 1.0;       (* Minimum conveyor speed, m/s *)
    Max_Load : REAL := 100.0;       (* Load scaling reference, kg *)
    Gain_FF : REAL := 0.02;         (* Feedforward gain, m/s per kg *)
    Min_Speed : REAL := 0.5;        (* Minimum safe speed, m/s *)
    Max_Speed : REAL := 2.0;        (* Maximum safe speed, m/s *)
    LastPredicted_Load : REAL;      (* Previous Predicted_Load value *)
    LastConveyor_Speed : REAL;      (* Previous Conveyor_Speed value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Feedforward: Conveyor_Speed = Base_Speed + Gain_FF * Predicted_Load
     - Base_Speed = 1.0 m/s, Gain_FF = 0.02 m/s per kg
   - Safety:
     - Validate Predicted_Load (0.0-200.0 kg)
     - Clamp Conveyor_Speed (0.5-2.0 m/s)
     - Set ControlError, default to Base_Speed on invalid input
   - Logs speed adjustments and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Conveyor_Speed := Base_Speed; (* Default to 1.0 m/s *)
ControlError := FALSE;

(* Step 2: Validate sensor input *)
IF NOT IS_VALID_REAL(Predicted_Load) OR Predicted_Load < 0.0 OR Predicted_Load > 200.0 THEN
    ControlError := TRUE;
    Conveyor_Speed := Base_Speed; (* Default to 1.0 m/s on error *)
    IF LogCount < 50 AND (Predicted_Load <> LastPredicted_Load OR 
                          ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 21:00:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Load Input=', 
            CONCAT(TO_STRING(Predicted_Load), ' kg'));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: Feedforward speed calculation *)
    Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load; (* Base_Speed=1.0, Gain_FF=0.02 *)

    (* Step 4: Clamp speed to operational limits *)
    IF Conveyor_Speed > Max_Speed THEN
        Conveyor_Speed := Max_Speed; (* Clamp to 2.0 m/s *)
    ELSIF Conveyor_Speed < Min_Speed THEN
        Conveyor_Speed := Min_Speed; (* Clamp to 0.5 m/s *)
    END_IF;

    (* Step 5: Log large load predictions for diagnostics *)
    IF Predicted_Load > 150.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 21:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Predicted Load=', 
            CONCAT(TO_STRING(Predicted_Load), ' kg'));
    END_IF;
END_IF;

(* Step 6: Log speed adjustments *)
IF Conveyor_Speed <> LastConveyor_Speed AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 21:00:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Conveyor Speed: ', 
        CONCAT(TO_STRING(Conveyor_Speed), ' m/s'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 21:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 21:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 7: Update last states for logging *)
LastPredicted_Load := Predicted_Load;
LastConveyor_Speed := Conveyor_Speed;
LastControlError := ControlError;

(* Step 8: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Feedforward control for conveyor speed based on predicted load.
   - Inputs:
     - Predicted_Load: REAL, predicted load (kg).
   - Outputs:
     - Conveyor_Speed: REAL, motor speed (m/s).
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Conveyor_Speed = 1.0 + 0.02 * Predicted_Load.
     - Clamp Conveyor_Speed (0.5-2.0 m/s).
     - Validate Predicted_Load (0.0-200.0 kg), default to 1.0 m/s on error.
   - Optimization:
     - Simple logic (~30 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates input (finite, 0.0-200.0 kg).
     - Clamps speed to safe range (0.5-2.0 m/s).
     - Logs errors for diagnostics.
   - Usage:
     - Conveyor system: Adjusts speed for load changes.
     - Example: Predicted_Load=50.0 → Conveyor_Speed=2.0 m/s.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
