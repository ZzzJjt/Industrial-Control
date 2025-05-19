(* IEC 61131-3 Structured Text Program: ChemicalMixingFeedforwardControl *)
(* Purpose: Implements feedforward control for chemical mixing ratio *)

PROGRAM ChemicalMixingFeedforwardControl
VAR
    (* Inputs *)
    Flow_B : REAL;                  (* Measured flow rate of Reactant B, L/min *)
    Concentration_B : REAL;         (* Concentration of Reactant B, fraction *)

    (* Outputs *)
    Flow_A_Setpoint : REAL;         (* Required flow rate of Reactant A, L/min *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Desired_Ratio : REAL := 2.0;    (* Target A:B mixing ratio *)
    Compensation_Factor : REAL := 1.0; (* Compensation for concentration *)
    Min_Flow_A : REAL := 0.0;       (* Minimum flow setpoint, L/min *)
    Max_Flow_A : REAL := 100.0;     (* Maximum flow setpoint, L/min *)
    LastFlow_B : REAL;              (* Previous Flow_B value *)
    LastConcentration_B : REAL;     (* Previous Concentration_B value *)
    LastFlow_A_Setpoint : REAL;     (* Previous Flow_A_Setpoint value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Feedforward: Flow_A_Setpoint = Desired_Ratio * Flow_B * Compensation_Factor
     - Desired_Ratio = 2.0 (2 parts A per 1 part B)
     - Compensation_Factor = 0.8 / Concentration_B (reference conc. 0.8)
   - Safety:
     - Validate Flow_B (0.0-100.0 L/min), Concentration_B (0.1-1.0)
     - Clamp Flow_A_Setpoint (0.0-100.0 L/min), Compensation_Factor (0.5-2.0)
     - Set ControlError, default Flow_A_Setpoint to 0.0 on invalid input
   - Logs setpoint adjustments and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Flow_A_Setpoint := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor inputs *)
IF NOT IS_VALID_REAL(Flow_B) OR NOT IS_VALID_REAL(Concentration_B) OR
   Flow_B < 0.0 OR Flow_B > 100.0 OR Concentration_B < 0.1 OR Concentration_B > 1.0 THEN
    ControlError := TRUE;
    Flow_A_Setpoint := 0.0;
    Compensation_Factor := 1.0;
    IF LogCount < 50 AND (Flow_B <> LastFlow_B OR 
                          Concentration_B <> LastConcentration_B OR 
                          ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 22:00:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Input - Flow_B=', 
            CONCAT(TO_STRING(Flow_B), CONCAT(' L/min, Conc_B=', 
            TO_STRING(Concentration_B))));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: Calculate compensation factor *)
    Compensation_Factor := 0.8 / Concentration_B; (* Reference conc. 0.8 *)
    Compensation_Factor := MIN(MAX(Compensation_Factor, 0.5), 2.0); (* Clamp to 0.5-2.0 *)

    (* Step 4: Feedforward flow calculation *)
    Flow_A_Setpoint := Desired_Ratio * Flow_B * Compensation_Factor; (* Desired_Ratio=2.0 *)

    (* Step 5: Clamp flow setpoint to operational limits *)
    Flow_A_Setpoint := MIN(MAX(Flow_A_Setpoint, Min_Flow_A), Max_Flow_A); (* 0.0-100.0 L/min *)

    (* Step 6: Log extreme concentration values for diagnostics *)
    IF (Concentration_B < 0.2 OR Concentration_B > 0.9) AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 22:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Extreme Concentration_B=', 
            TO_STRING(Concentration_B));
    END_IF;
END_IF;

(* Step 7: Log setpoint adjustments *)
IF Flow_A_Setpoint <> LastFlow_A_Setpoint AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 22:00:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Flow A Setpoint: ', 
        CONCAT(TO_STRING(Flow_A_Setpoint), ' L/min'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 22:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 22:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
LastFlow_B := Flow_B;
LastConcentration_B := Concentration_B;
LastFlow_A_Setpoint := Flow_A_Setpoint;
LastControlError := ControlError;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Feedforward control for chemical mixing, adjusts Reactant A flow for 2:1 A:B ratio.
   - Inputs:
     - Flow_B: REAL, Reactant B flow rate (L/min).
     - Concentration_B: REAL, Reactant B concentration (fraction).
   - Outputs:
     - Flow_A_Setpoint: REAL, Reactant A flow setpoint (L/min).
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Flow_A_Setpoint = 2.0 * Flow_B * (0.8 / Concentration_B).
     - Clamp Flow_A_Setpoint (0.0-100.0 L/min), Compensation_Factor (0.5-2.0).
     - Validate Flow_B (0.0-100.0 L/min), Concentration_B (0.1-1.0).
     - Default Flow_A_Setpoint to 0.0 on error.
   - Optimization:
     - Simple logic (~40 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates inputs (finite, ranges).
     - Clamps setpoint to safe range.
     - Logs errors for diagnostics.
   - Usage:
     - Chemical mixing: Maintains 2:1 A:B ratio, adjusts for concentration.
     - Example: Flow_B=10.0, Concentration_B=0.8 → Flow_A_Setpoint=20.0 L/min.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
