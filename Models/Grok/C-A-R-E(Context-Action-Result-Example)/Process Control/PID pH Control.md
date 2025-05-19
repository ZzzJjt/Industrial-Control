(* IEC 61131-3 Structured Text Program: pHRegulationPIDControl *)
(* Purpose: Implements PID control for pH regulation in chemical process *)

PROGRAM pHRegulationPIDControl
VAR
    (* Inputs *)
    pH_PV : REAL;                   (* Measured pH value *)

    (* Outputs *)
    Dosing_Output : REAL;           (* Acid/base dosing control signal, 0.0-100.0% *)
    Error : REAL;                   (* Control error, pH units *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    pH_SP : REAL := 7.0;            (* pH setpoint *)
    Kp : REAL := 2.5;               (* Proportional gain, % dosing/pH *)
    Ki : REAL := 0.6;               (* Integral gain, % dosing/(pH·s) *)
    Kd : REAL := 0.3;               (* Derivative gain, % dosing/(pH/s) *)
    Sample_Time : REAL := 0.1;      (* Sampling time, 100 ms *)
    Dosing_Min : REAL := 0.0;       (* Minimum dosing output, % *)
    Dosing_Max : REAL := 100.0;     (* Maximum dosing output, % *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative *)
    Integral : REAL := 0.0;         (* Integral term accumulation *)
    Derivative : REAL;              (* Derivative term *)
    Last_pH_PV : REAL;              (* Previous pH_PV value *)
    LastDosing_Output : REAL;       (* Previous Dosing_Output value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - PID: Maintains pH 7.0
     - Error = pH_SP - pH_PV
     - Integral += Error * 0.1 (100 ms sample time)
     - Derivative = (Error - Prev_Error) / 0.1
     - Dosing_Output = Kp * Error + Ki * Integral + Kd * Derivative
   - Safety:
     - Validate pH_PV (0.0-14.0)
     - Clamp Dosing_Output (0.0-100.0%)
     - Anti-windup: Freeze Integral when output clamped
     - Set ControlError, default Dosing_Output to 0.0 on invalid input
   - Logs dosing output changes and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Dosing_Output := 0.0;
Error := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor input *)
IF NOT IS_VALID_REAL(pH_PV) OR pH_PV < 0.0 OR pH_PV > 14.0 THEN
    ControlError := TRUE;
    Dosing_Output := 0.0;
    Integral := 0.0; (* Reset integral on error *)
    IF LogCount < 50 AND (pH_PV <> Last_pH_PV OR ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:29:00'; (* Current date/time *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid pH Input=', 
            CONCAT(TO_STRING(pH_PV), ''));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: PID control logic *)
    Error := pH_SP - pH_PV; (* Error in pH units *)
    Derivative := (Error - Prev_Error) / Sample_Time; (* pH/s *)

    (* Anti-windup: Update integral only if output not clamped *)
    Dosing_Output := Kp * Error + Ki * Integral + Kd * Derivative; (* Preliminary output *)
    IF Dosing_Output >= Dosing_Min AND Dosing_Output <= Dosing_Max THEN
        Integral := Integral + Error * Sample_Time; (* Update integral, pH·s *)
    END_IF;

    (* Final output calculation *)
    Dosing_Output := Kp * Error + Ki * Integral + Kd * Derivative;

    (* Step 4: Clamp dosing output *)
    IF Dosing_Output > Dosing_Max THEN
        Dosing_Output := Dosing_Max; (* Clamp to 100.0% *)
    ELSIF Dosing_Output < Dosing_Min THEN
        Dosing_Output := Dosing_Min; (* Clamp to 0.0% *)
    END_IF;

    (* Step 5: Update previous error *)
    Prev_Error := Error;

    (* Step 6: Log large errors for diagnostics *)
    IF ABS(Error) > 2.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:29:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Error=', 
            CONCAT(TO_STRING(Error), ' pH'));
    END_IF;
END_IF;

(* Step 7: Log dosing output changes *)
IF Dosing_Output <> LastDosing_Output AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:29:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Dosing Output: ', 
        CONCAT(TO_STRING(Dosing_Output), '%'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:29:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:29:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
Last_pH_PV := pH_PV;
LastDosing_Output := Dosing_Output;
LastControlError := ControlError;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: PID control for pH regulation, maintains pH 7.0 in chemical process.
   - Inputs:
     - pH_PV: REAL, measured pH value.
   - Outputs:
     - Dosing_Output: REAL, dosing control signal (0.0-100.0%).
     - Error: REAL, control error (pH units).
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Error = 7.0 - pH_PV.
     - Integral += Error * 0.1, Derivative = (Error - Prev_Error) / 0.1.
     - Dosing_Output = 2.5 * Error + 0.6 * Integral + 0.3 * Derivative.
     - Clamp Dosing_Output (0.0-100.0%), anti-windup on Integral.
     - Validate pH_PV (0.0-14.0).
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates input (finite, 0.0-14.0).
     - Clamps output to safe range.
     - Anti-windup prevents integral overshoot.
     - Logs errors for diagnostics.
   - Usage:
     - Chemical process: Maintains pH 7.0, prevents corrosion/inconsistencies.
     - Example: pH_PV=6.8 → Dosing_Output adjusts to correct.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp uses current date/time; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
