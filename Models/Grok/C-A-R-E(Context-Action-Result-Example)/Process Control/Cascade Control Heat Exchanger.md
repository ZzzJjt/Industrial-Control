(* IEC 61131-3 Structured Text Program: HeatExchangerCascadeControl *)
(* Purpose: Implements cascade control for heat exchanger temperature *)

PROGRAM HeatExchangerCascadeControl
VAR
    (* Inputs *)
    Temp_PV : REAL;                 (* Measured outlet temperature, °C *)
    Flow_PV : REAL;                 (* Measured flow rate, LPM *)

    (* Outputs *)
    Flow_Output : REAL;             (* Valve control signal, 0.0-100.0% *)
    Flow_SP : REAL;                 (* Flow setpoint, LPM *)
    Temp_Error : REAL;              (* Temperature error, °C *)
    Flow_Error : REAL;              (* Flow error, LPM *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Temp_SP : REAL := 85.0;         (* Temperature setpoint, °C *)
    Kp_Outer : REAL := 1.0;         (* Outer loop proportional gain *)
    Kp_Inner : REAL := 2.0;         (* Inner loop proportional gain *)
    Temp_Output : REAL;             (* Outer loop output, LPM *)
    LastTemp_PV : REAL;             (* Previous Temp_PV value *)
    LastFlow_PV : REAL;             (* Previous Flow_PV value *)
    LastFlow_Output : REAL;         (* Previous Flow_Output value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Outer Loop: Compares Temp_SP (85.0°C) to Temp_PV, sets Flow_SP
     - Temp_Error = Temp_SP - Temp_PV
     - Flow_SP = Kp_Outer * Temp_Error (Kp_Outer = 1.0)
   - Inner Loop: Adjusts Flow_Output to match Flow_SP
     - Flow_Error = Flow_SP - Flow_PV
     - Flow_Output = Kp_Inner * Flow_Error (Kp_Inner = 2.0)
   - Safety:
     - Validate Temp_PV (0.0-150.0°C), Flow_PV (0.0-100.0 LPM)
     - Clamp Flow_SP (0.0-100.0 LPM), Flow_Output (0.0-100.0%)
     - Set ControlError, disable Flow_Output on invalid inputs
   - Logs control actions and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Flow_Output := 0.0;
Flow_SP := 0.0;
Temp_Error := 0.0;
Flow_Error := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor inputs *)
IF NOT IS_VALID_REAL(Temp_PV) OR NOT IS_VALID_REAL(Flow_PV) OR
   Temp_PV < 0.0 OR Temp_PV > 150.0 OR Flow_PV < 0.0 OR Flow_PV > 100.0 THEN
    ControlError := TRUE;
    Flow_Output := 0.0;
    Flow_SP := 0.0;
    IF LogCount < 50 AND (Temp_PV <> LastTemp_PV OR Flow_PV <> LastFlow_PV OR 
                          ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 18:30:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Input - Temp=', 
            CONCAT(TO_STRING(Temp_PV), CONCAT(' °C, Flow=', 
            CONCAT(TO_STRING(Flow_PV), ' LPM'))));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: Outer loop - Temperature to flow setpoint *)
    Temp_Error := Temp_SP - Temp_PV;
    Temp_Output := Kp_Outer * Temp_Error; (* Kp_Outer = 1.0, °C to LPM *)
    Flow_SP := MIN(MAX(Temp_Output, 0.0), 100.0); (* Clamp to 0.0-100.0 LPM *)

    (* Step 4: Inner loop - Flow to valve output *)
    Flow_Error := Flow_SP - Flow_PV;
    Flow_Output := Kp_Inner * Flow_Error; (* Kp_Inner = 2.0, LPM to % *)
    Flow_Output := MIN(MAX(Flow_Output, 0.0), 100.0); (* Clamp to 0.0-100.0% *)

    (* Step 5: Log large errors for diagnostics *)
    IF ABS(Temp_Error) > 50.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 18:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Temp Error=', 
            CONCAT(TO_STRING(Temp_Error), ' °C'));
    END_IF;
    IF ABS(Flow_Error) > 50.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 18:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Flow Error=', 
            CONCAT(TO_STRING(Flow_Error), ' LPM'));
    END_IF;
END_IF;

(* Step 6: Log control actions *)
IF Flow_Output <> LastFlow_Output AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 18:30:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Flow Output: ', 
        CONCAT(TO_STRING(Flow_Output), '%'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 18:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 18:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 7: Update last states for logging *)
LastTemp_PV := Temp_PV;
LastFlow_PV := Flow_PV;
LastFlow_Output := Flow_Output;
LastControlError := ControlError;

(* Step 8: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Cascade control for heat exchanger, outer temp loop sets flow setpoint, inner flow loop adjusts valve.
   - Inputs:
     - Temp_PV: REAL, measured temperature (°C).
     - Flow_PV: REAL, measured flow rate (LPM).
   - Outputs:
     - Flow_Output: REAL, valve signal (0.0-100.0%).
     - Flow_SP: REAL, flow setpoint (LPM).
     - Temp_Error, Flow_Error: REAL, control errors.
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Outer: Temp_Error = 85.0 - Temp_PV, Flow_SP = Kp_Outer * Temp_Error (Kp_Outer=1.0).
     - Inner: Flow_Error = Flow_SP - Flow_PV, Flow_Output = Kp_Inner * Flow_Error (Kp_Inner=2.0).
     - Clamp Flow_SP (0.0-100.0 LPM), Flow_Output (0.0-100.0%).
     - Safety: Validate inputs (0.0-150.0°C, 0.0-100.0 LPM), disable output on fault.
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates inputs (finite, ranges).
     - Clamps outputs to safe ranges.
     - Logs errors for diagnostics.
   - Usage:
     - Heat exchanger: Maintains 85.0°C via flow control.
     - Example: Temp_PV=83.0 → Flow_SP=2.0 LPM; Flow_PV=1.5 → Flow_Output=1.0%.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
