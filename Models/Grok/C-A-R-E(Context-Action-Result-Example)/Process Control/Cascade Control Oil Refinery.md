(* IEC 61131-3 Structured Text Program: OilVesselCascadeControl *)
(* Purpose: Implements cascade control for oil refinery vessel pressure *)

PROGRAM OilVesselCascadeControl
VAR
    (* Inputs *)
    Pressure_PV : REAL;             (* Measured vessel pressure, bar *)
    Flow_PV : REAL;                 (* Measured oil inflow rate, L/min *)

    (* Outputs *)
    Flow_Output : REAL;             (* Valve control signal, 0.0-100.0% *)
    Flow_SP : REAL;                 (* Flow setpoint, L/min *)
    Pressure_Error : REAL;          (* Pressure error, bar *)
    Flow_Error : REAL;              (* Flow error, L/min *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Pressure_SP : REAL := 12.0;     (* Pressure setpoint, bar *)
    Kp_Outer : REAL := 1.2;         (* Outer loop proportional gain *)
    Kp_Inner : REAL := 2.5;         (* Inner loop proportional gain *)
    LastPressure_PV : REAL;         (* Previous Pressure_PV value *)
    LastFlow_PV : REAL;             (* Previous Flow_PV value *)
    LastFlow_Output : REAL;         (* Previous Flow_Output value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Outer Loop: Compares Pressure_SP (12.0 bar) to Pressure_PV, sets Flow_SP
     - Pressure_Error = Pressure_SP - Pressure_PV
     - Flow_SP = Kp_Outer * Pressure_Error (Kp_Outer=1.2)
   - Inner Loop: Adjusts Flow_Output to match Flow_SP
     - Flow_Error = Flow_SP - Flow_PV
     - Flow_Output = Kp_Inner * Flow_Error (Kp_Inner=2.5)
   - Safety:
     - Validate Pressure_PV (0.0-20.0 bar), Flow_PV (0.0-200.0 L/min)
     - Clamp Flow_SP (0.0-200.0 L/min), Flow_Output (0.0-100.0%)
     - Set ControlError, disable Flow_Output on invalid inputs
   - Logs control actions and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Flow_Output := 0.0;
Flow_SP := 0.0;
Pressure_Error := 0.0;
Flow_Error := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor inputs *)
IF NOT IS_VALID_REAL(Pressure_PV) OR NOT IS_VALID_REAL(Flow_PV) OR
   Pressure_PV < 0.0 OR Pressure_PV > 20.0 OR Flow_PV < 0.0 OR Flow_PV > 200.0 THEN
    ControlError := TRUE;
    Flow_Output := 0.0;
    Flow_SP := 0.0;
    IF LogCount < 50 AND (Pressure_PV <> LastPressure_PV OR Flow_PV <> LastFlow_PV OR 
                          ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 20:00:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Input - Pressure=', 
            CONCAT(TO_STRING(Pressure_PV), CONCAT(' bar, Flow=', 
            CONCAT(TO_STRING(Flow_PV), ' L/min'))));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: Outer loop - Pressure to flow setpoint *)
    Pressure_Error := Pressure_SP - Pressure_PV;
    Flow_SP := Kp_Outer * Pressure_Error; (* Kp_Outer = 1.2, bar to L/min *)
    Flow_SP := MIN(MAX(Flow_SP, 0.0), 200.0); (* Clamp to 0.0-200.0 L/min *)

    (* Step 4: Inner loop - Flow to valve output *)
    Flow_Error := Flow_SP - Flow_PV;
    Flow_Output := Kp_Inner * Flow_Error; (* Kp_Inner = 2.5, L/min to % *)
    Flow_Output := MIN(MAX(Flow_Output, 0.0), 100.0); (* Clamp to 0.0-100.0% *)

    (* Step 5: Log large errors for diagnostics *)
    IF ABS(Pressure_Error) > 10.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 20:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Pressure Error=', 
            CONCAT(TO_STRING(Pressure_Error), ' bar'));
    END_IF;
    IF ABS(Flow_Error) > 50.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 20:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Flow Error=', 
            CONCAT(TO_STRING(Flow_Error), ' L/min'));
    END_IF;
END_IF;

(* Step 6: Log control actions *)
IF Flow_Output <> LastFlow_Output AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 20:00:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Flow Output: ', 
        CONCAT(TO_STRING(Flow_Output), '%'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 20:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 20:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 7: Update last states for logging *)
LastPressure_PV := Pressure_PV;
LastFlow_PV := Flow_PV;
LastFlow_Output := Flow_Output;
LastControlError := ControlError;

(* Step 8: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Cascade control for oil refinery vessel, outer pressure loop sets flow setpoint, inner flow loop adjusts valve.
   - Inputs:
     - Pressure_PV: REAL, measured pressure (bar).
     - Flow_PV: REAL, measured flow rate (L/min).
   - Outputs:
     - Flow_Output: REAL, valve signal (0.0-100.0%).
     - Flow_SP: REAL, flow setpoint (L/min).
     - Pressure_Error, Flow_Error: REAL, control errors.
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Outer: Pressure_Error = 12.0 - Pressure_PV, Flow_SP = Kp_Outer * Pressure_Error (Kp_Outer=1.2).
     - Inner: Flow_Error = Flow_SP - Flow_PV, Flow_Output = Kp_Inner * Flow_Error (Kp_Inner=2.5).
     - Clamp Flow_SP (0.0-200.0 L/min), Flow_Output (0.0-100.0%).
     - Safety: Validate inputs (0.0-20.0 bar, 0.0-200.0 L/min), disable output on fault.
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates inputs (finite, ranges).
     - Clamps outputs to safe ranges.
     - Logs errors for diagnostics.
   - Usage:
     - Oil refinery vessel: Maintains 12.0 bar via flow control.
     - Example: Pressure_PV=11.5 → Flow_SP=0.6 L/min; Flow_PV=0.4 → Flow_Output=0.5%.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
