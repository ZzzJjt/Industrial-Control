(* IEC 61131-3 Structured Text Program: ReactorPressurePIDControl *)
(* Purpose: Implements PID control for chemical reactor pressure *)

PROGRAM ReactorPressurePIDControl
VAR
    (* Inputs *)
    Pressure_PV : REAL;             (* Measured reactor pressure, bar *)

    (* Outputs *)
    Valve_Output : REAL;            (* Pressure control valve signal, 0.0-100.0% *)
    Error : REAL;                   (* Control error, bar *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Pressure_SP : REAL := 5.0;      (* Pressure setpoint, bar *)
    Kp : REAL := 2.0;               (* Proportional gain, % valve/bar *)
    Ki : REAL := 0.8;               (* Integral gain, % valve/(bar·s) *)
    Kd : REAL := 0.3;               (* Derivative gain, % valve/(bar/s) *)
    Sample_Time : REAL := 0.1;      (* Sampling time, 100 ms *)
    Valve_Min : REAL := 0.0;        (* Minimum valve position, % *)
    Valve_Max : REAL := 100.0;      (* Maximum valve position, % *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative *)
    Integral : REAL := 0.0;         (* Integral term accumulation *)
    Derivative : REAL;              (* Derivative term *)
    LastPressure_PV : REAL;         (* Previous Pressure_PV value *)
    LastValve_Output : REAL;        (* Previous Valve_Output value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - PID: Maintains 5.0 bar pressure
     - Error = Pressure_SP - Pressure_PV
     - Integral += Error * 0.1 (100 ms sample time)
     - Derivative = (Error - Prev_Error) / 0.1
     - Valve_Output = Kp * Error + Ki * Integral + Kd * Derivative
   - Safety:
     - Validate Pressure_PV (0.0-10.0 bar)
     - Clamp Valve_Output (0.0-100.0%)
     - Anti-windup: Freeze Integral when output clamped
     - Set ControlError, default Valve_Output to 0.0 on invalid input
   - Logs valve output changes and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Valve_Output := 0.0;
Error := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor input *)
IF NOT IS_VALID_REAL(Pressure_PV) OR Pressure_PV < 0.0 OR Pressure_PV > 10.0 THEN
    ControlError := TRUE;
    Valve_Output := 0.0;
    Integral := 0.0; (* Reset integral on error *)
    IF LogCount < 50 AND (Pressure_PV <> LastPressure_PV OR ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:25:00'; (* Current date/time *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Pressure Input=', 
            CONCAT(TO_STRING(Pressure_PV), ' bar'));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: PID control logic *)
    Error := Pressure_SP - Pressure_PV; (* Error in bar *)
    Derivative := (Error - Prev_Error) / Sample_Time; (* bar/s *)

    (* Anti-windup: Update integral only if output not clamped *)
    Valve_Output := Kp * Error + Ki * Integral + Kd * Derivative; (* Preliminary output *)
    IF Valve_Output >= Valve_Min AND Valve_Output <= Valve_Max THEN
        Integral := Integral + Error * Sample_Time; (* Update integral, bar·s *)
    END_IF;

    (* Final output calculation *)
    Valve_Output := Kp * Error + Ki * Integral + Kd * Derivative;

    (* Step 4: Clamp valve output *)
    IF Valve_Output > Valve_Max THEN
        Valve_Output := Valve_Max; (* Clamp to 100.0% *)
    ELSIF Valve_Output < Valve_Min THEN
        Valve_Output := Valve_Min; (* Clamp to 0.0% *)
    END_IF;

    (* Step 5: Update previous error *)
    Prev_Error := Error;

    (* Step 6: Log large errors for diagnostics *)
    IF ABS(Error) > 2.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:25:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Error=', 
            CONCAT(TO_STRING(Error), ' bar'));
    END_IF;
END_IF;

(* Step 7: Log valve output changes *)
IF Valve_Output <> LastValve_Output AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:25:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Valve Output: ', 
        CONCAT(TO_STRING(Valve_Output), '%'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:25:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:25:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
LastPressure_PV := Pressure_PV;
LastValve_Output := Valve_Output;
LastControlError := ControlError;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: PID control for chemical reactor pressure, maintains 5.0 bar.
   - Inputs:
     - Pressure_PV: REAL, measured pressure (bar).
   - Outputs:
     - Valve_Output: REAL, valve signal (0.0-100.0%).
     - Error: REAL, control error (bar).
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Error = 5.0 - Pressure_PV.
     - Integral += Error * 0.1, Derivative = (Error - Prev_Error) / 0.1.
     - Valve_Output = 2.0 * Error + 0.8 * Integral + 0.3 * Derivative.
     - Clamp Valve_Output (0.0-100.0%), anti-windup on Integral.
     - Validate Pressure_PV (0.0-10.0 bar).
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates input (finite, 0.0-10.0 bar).
     - Clamps output to safe range.
     - Anti-windup prevents integral overshoot.
     - Logs errors for diagnostics.
   - Usage:
     - Chemical reactor: Maintains 5.0 bar, prevents unsafe pressures.
     - Example: Pressure_PV=4.8 → Valve_Output adjusts to correct.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp uses current date/time; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
