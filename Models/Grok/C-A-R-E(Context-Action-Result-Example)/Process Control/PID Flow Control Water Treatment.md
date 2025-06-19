(* IEC 61131-3 Structured Text Program: ChlorineDosingPIDControl *)
(* Purpose: Implements PID control for chlorine dosing in water treatment *)

PROGRAM ChlorineDosingPIDControl
VAR
    (* Inputs *)
    FlowRate : REAL;                (* Current water flow rate, L/min *)
    Dosing_PV : REAL;               (* Measured chlorine concentration, ppm *)

    (* Outputs *)
    Dosing_Output : REAL;           (* Dosing pump control signal, 0.0-10.0% *)
    Error : REAL;                   (* Control error, ppm *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Dosing_SP : REAL := 3.0;        (* Chlorine concentration setpoint, ppm *)
    Kp : REAL := 2.0;               (* Proportional gain, %/ppm *)
    Ki : REAL := 0.5;               (* Integral gain, %/(ppm·s) *)
    Kd : REAL := 0.1;               (* Derivative gain, %/(ppm/s) *)
    Sample_Time : REAL := 0.1;      (* Sampling time, 100 ms *)
    Max_Dose : REAL := 10.0;        (* Maximum dosing rate, % *)
    Min_Dose : REAL := 0.0;         (* Minimum dosing rate, % *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative *)
    Integral : REAL := 0.0;         (* Integral term accumulation *)
    Derivative : REAL;              (* Derivative term *)
    LastFlowRate : REAL;            (* Previous FlowRate value *)
    LastDosing_PV : REAL;           (* Previous Dosing_PV value *)
    LastDosing_Output : REAL;       (* Previous Dosing_Output value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - PID: Maintains 3.0 ppm chlorine concentration
     - Error = Dosing_SP - Dosing_PV
     - Integral += Error * 0.1 (100 ms sample time)
     - Derivative = (Error - Prev_Error) / 0.1
     - Dosing_Output = Kp * Error + Ki * Integral + Kd * Derivative
   - Safety:
     - Validate FlowRate (0.0-1000.0 L/min), Dosing_PV (0.0-10.0 ppm)
     - Clamp Dosing_Output (0.0-10.0%)
     - Anti-windup: Freeze Integral when output clamped
     - Set ControlError, default Dosing_Output to 0.0 on invalid input
   - Logs dosing adjustments and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Dosing_Output := 0.0;
Error := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor inputs *)
IF NOT IS_VALID_REAL(FlowRate) OR NOT IS_VALID_REAL(Dosing_PV) OR
   FlowRate < 0.0 OR FlowRate > 1000.0 OR Dosing_PV < 0.0 OR Dosing_PV > 10.0 THEN
    ControlError := TRUE;
    Dosing_Output := 0.0;
    Integral := 0.0; (* Reset integral on error *)
    IF LogCount < 50 AND (FlowRate <> LastFlowRate OR 
                          Dosing_PV <> LastDosing_PV OR 
                          ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 23:00:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Input - Flow=', 
            CONCAT(TO_STRING(FlowRate), CONCAT(' L/min, Dosing_PV=', 
            TO_STRING(Dosing_PV))));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: PID control logic *)
    Error := Dosing_SP - Dosing_PV; (* Error in ppm *)
    Derivative := (Error - Prev_Error) / Sample_Time; (* ppm/s *)

    (* Anti-windup: Update integral only if output not clamped *)
    Dosing_Output := Kp * Error + Ki * Integral + Kd * Derivative; (* Preliminary output *)
    IF Dosing_Output >= Min_Dose AND Dosing_Output <= Max_Dose THEN
        Integral := Integral + Error * Sample_Time; (* Update integral, ppm·s *)
    END_IF;

    (* Final output calculation *)
    Dosing_Output := Kp * Error + Ki * Integral + Kd * Derivative;

    (* Step 4: Clamp dosing output *)
    IF Dosing_Output > Max_Dose THEN
        Dosing_Output := Max_Dose; (* Clamp to 10.0% *)
    ELSIF Dosing_Output < Min_Dose THEN
        Dosing_Output := Min_Dose; (* Clamp to 0.0% *)
    END_IF;

    (* Step 5: Update previous error *)
    Prev_Error := Error;

    (* Step 6: Log large errors for diagnostics *)
    IF ABS(Error) > 5.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 23:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Error=', 
            CONCAT(TO_STRING(Error), ' ppm'));
    END_IF;
END_IF;

(* Step 7: Log dosing output changes *)
IF Dosing_Output <> LastDosing_Output AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 23:00:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Dosing Output: ', 
        CONCAT(TO_STRING(Dosing_Output), '%'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 23:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 23:00:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
LastFlowRate := FlowRate;
LastDosing_PV := Dosing_PV;
LastDosing_Output := Dosing_Output;
LastControlError := ControlError;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: PID control for chlorine dosing, maintains 3.0 ppm in water treatment.
   - Inputs:
     - FlowRate: REAL, water flow rate (L/min).
     - Dosing_PV: REAL, measured chlorine concentration (ppm).
   - Outputs:
     - Dosing_Output: REAL, dosing pump signal (0.0-10.0%).
     - Error: REAL, control error (ppm).
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Error = 3.0 - Dosing_PV.
     - Integral += Error * 0.1, Derivative = (Error - Prev_Error) / 0.1.
     - Dosing_Output = 2.0 * Error + 0.5 * Integral + 0.1 * Derivative.
     - Clamp Dosing_Output (0.0-10.0%), anti-windup on Integral.
     - Validate FlowRate (0.0-1000.0 L/min), Dosing_PV (0.0-10.0 ppm).
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates inputs (finite, ranges).
     - Clamps output to safe range.
     - Anti-windup prevents integral overshoot.
     - Logs errors for diagnostics.
   - Usage:
     - Water treatment: Maintains 3.0 ppm chlorine under varying flow.
     - Example: Dosing_PV=2.5 → Dosing_Output adjusts to correct.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
