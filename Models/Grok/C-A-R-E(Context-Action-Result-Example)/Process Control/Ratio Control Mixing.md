(* IEC 61131-3 Structured Text Program: RatioControl *)
(* Purpose: Implements ratio control for 2:1 A:B mixing in chemical blending *)

PROGRAM RatioControl
VAR
    (* Inputs *)
    Flow_A_PV : REAL;               (* Measured flow rate of reactant A, L/min *)
    Flow_B_PV : REAL;               (* Measured flow rate of reactant B, L/min *)

    (* Outputs *)
    Flow_B_SP : REAL;               (* Setpoint flow rate for reactant B, L/min *)
    Actual_Ratio : REAL;            (* Current A:B flow ratio *)
    Ratio_Error : REAL;             (* Deviation from setpoint ratio *)
    Ratio_Alarm : BOOL := FALSE;    (* TRUE if ratio deviation exceeds tolerance *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Ratio_Setpoint : REAL := 2.0;   (* Desired A:B ratio, 2:1 *)
    Tolerance : REAL := 0.05;       (* Acceptable ratio deviation, ±5% *)
    Min_Flow : REAL := 0.0;         (* Minimum flow rate, L/min *)
    Max_Flow : REAL := 100.0;       (* Maximum flow rate, L/min *)
    Last_Flow_A_PV : REAL;          (* Previous Flow_A_PV value *)
    Last_Flow_B_PV : REAL;          (* Previous Flow_B_PV value *)
    Last_Flow_B_SP : REAL;          (* Previous Flow_B_SP value *)
    Last_Ratio_Alarm : BOOL;        (* Previous Ratio_Alarm state *)
    Last_ControlError : BOOL;       (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Ratio Control: Maintains 2:1 A:B flow ratio
     - Actual_Ratio = Flow_A_PV / Flow_B_PV if Flow_B_PV > 0.01, else 0.0
     - Flow_B_SP = Flow_A_PV / 2.0
     - Ratio_Error = Actual_Ratio - 2.0
     - Ratio_Alarm = TRUE if ABS(Ratio_Error) > 0.05
   - Safety:
     - Validate Flow_A_PV, Flow_B_PV (0.0-100.0 L/min)
     - Clamp Flow_B_SP (0.0-100.0 L/min)
     - Set ControlError, default Flow_B_SP to 0.0 on invalid input
   - Logs setpoint updates, ratio alarms, and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Flow_B_SP := 0.0;
Actual_Ratio := 0.0;
Ratio_Error := 0.0;
Ratio_Alarm := FALSE;
ControlError := FALSE;

(* Step 2: Validate sensor inputs *)
IF NOT IS_VALID_REAL(Flow_A_PV) OR NOT IS_VALID_REAL(Flow_B_PV) OR
   Flow_A_PV < Min_Flow OR Flow_A_PV > Max_Flow OR 
   Flow_B_PV < Min_Flow OR Flow_B_PV > Max_Flow THEN
    ControlError := TRUE;
    Flow_B_SP := 0.0;
    Actual_Ratio := 0.0;
    Ratio_Error := 0.0;
    Ratio_Alarm := FALSE;
    IF LogCount < 50 AND (Flow_A_PV <> Last_Flow_A_PV OR 
                          Flow_B_PV <> Last_Flow_B_PV OR 
                          ControlError <> Last_ControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:32:00'; (* Current date/time *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Input - Flow_A=', 
            CONCAT(TO_STRING(Flow_A_PV), CONCAT(' L/min, Flow_B=', 
            CONCAT(TO_STRING(Flow_B_PV), ' L/min'))));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: Calculate actual ratio *)
    IF Flow_B_PV > 0.01 THEN (* Avoid division by zero *)
        Actual_Ratio := Flow_A_PV / Flow_B_PV;
    ELSE
        Actual_Ratio := 0.0;
        IF LogCount < 50 AND Flow_B_PV <> Last_Flow_B_PV THEN
            LogCount := LogCount + 1;
            Timestamp := '2025-05-19 12:32:00';
            DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Low Flow_B_PV=', 
                CONCAT(TO_STRING(Flow_B_PV), ' L/min, Ratio Unreliable'));
        END_IF;
    END_IF;

    (* Step 4: Calculate desired flow for reactant B *)
    Flow_B_SP := Flow_A_PV / Ratio_Setpoint; (* Ratio_Setpoint = 2.0 *)

    (* Step 5: Clamp Flow_B_SP to operational limits *)
    Flow_B_SP := MIN(MAX(Flow_B_SP, Min_Flow), Max_Flow); (* 0.0-100.0 L/min *)

    (* Step 6: Check ratio deviation *)
    Ratio_Error := Actual_Ratio - Ratio_Setpoint;
    Ratio_Alarm := ABS(Ratio_Error) > Tolerance; (* Tolerance = 0.05 *)
END_IF;

(* Step 7: Log setpoint and alarm state changes *)
IF Flow_B_SP <> Last_Flow_B_SP AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:32:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Flow B Setpoint: ', 
        CONCAT(TO_STRING(Flow_B_SP), ' L/min'));
END_IF;
IF Ratio_Alarm AND NOT Last_Ratio_Alarm AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:32:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Ratio Alarm Active: Actual_Ratio=', 
        CONCAT(TO_STRING(Actual_Ratio), ', Error=', 
        CONCAT(TO_STRING(Ratio_Error), '')));
END_IF;
IF NOT Ratio_Alarm AND Last_Ratio_Alarm AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:32:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Ratio Alarm Cleared');
END_IF;
IF ControlError AND NOT Last_ControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:32:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND Last_ControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:32:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
Last_Flow_A_PV := Flow_A_PV;
Last_Flow_B_PV := Flow_B_PV;
Last_Flow_B_SP := Flow_B_SP;
Last_Ratio_Alarm := Ratio_Alarm;
Last_ControlError := ControlError;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Ratio control for 2:1 A:B mixing in chemical blending.
   - Inputs:
     - Flow_A_PV: REAL, measured flow of reactant A (L/min).
     - Flow_B_PV: REAL, measured flow of reactant B (L/min).
   - Outputs:
     - Flow_B_SP: REAL, setpoint for reactant B (L/min).
     - Actual_Ratio: REAL, current A:B ratio.
     - Ratio_Error: REAL, deviation from setpoint ratio.
     - Ratio_Alarm: BOOL, TRUE if ratio deviates >5%.
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Actual_Ratio = Flow_A_PV / Flow_B_PV if Flow_B_PV > 0.01, else 0.0.
     - Flow_B_SP = Flow_A_PV / 2.0.
     - Ratio_Error = Actual_Ratio - 2.0, Ratio_Alarm if ABS(Ratio_Error) > 0.05.
     - Clamp Flow_B_SP (0.0-100.0 L/min).
     - Validate Flow_A_PV, Flow_B_PV (0.0-100.0 L/min).
   - Optimization:
     - Simple logic (~40 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates inputs (finite, 0.0-100.0 L/min).
     - Clamps setpoint to safe range.
     - Handles low Flow_B_PV to avoid division by zero.
     - Logs alarms and errors for diagnostics.
   - Usage:
     - Chemical blending: Maintains 2:1 A:B ratio, adjusts to flow changes.
     - Example: Flow_A_PV=10.0 → Flow_B_SP=5.0 L/min.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp uses current date/time; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
