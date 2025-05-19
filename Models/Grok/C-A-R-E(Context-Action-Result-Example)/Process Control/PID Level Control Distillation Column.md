(* IEC 61131-3 Structured Text Program: DistillationLevelPIDControl *)
(* Purpose: Implements PID control for liquid level in distillation column *)

PROGRAM DistillationLevelPIDControl
VAR
    (* Inputs *)
    Level_PV : REAL;                (* Measured liquid level, % *)

    (* Outputs *)
    Valve_Position : REAL;          (* Inlet valve control signal, 0.0-100.0% *)
    Error : REAL;                   (* Control error, % *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    Level_SP : REAL := 60.0;        (* Liquid level setpoint, % *)
    Kp : REAL := 1.5;               (* Proportional gain, % valve/% level *)
    Ki : REAL := 0.4;               (* Integral gain, % valve/(%·s) *)
    Kd : REAL := 0.2;               (* Derivative gain, % valve/(%/s) *)
    Sample_Time : REAL := 0.1;      (* Sampling time, 100 ms *)
    Valve_Min : REAL := 0.0;        (* Minimum valve position, % *)
    Valve_Max : REAL := 100.0;      (* Maximum valve position, % *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative *)
    Integral : REAL := 0.0;         (* Integral term accumulation *)
    Derivative : REAL;              (* Derivative term *)
    LastLevel_PV : REAL;            (* Previous Level_PV value *)
    LastValve_Position : REAL;      (* Previous Valve_Position value *)
    LastControlError : BOOL;        (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - PID: Maintains 60.0% liquid level
     - Error = Level_SP - Level_PV
     - Integral += Error * 0.1 (100 ms sample time)
     - Derivative = (Error - Prev_Error) / 0.1
     - Valve_Position = Kp * Error + Ki * Integral + Kd * Derivative
   - Safety:
     - Validate Level_PV (0.0-100.0%)
     - Clamp Valve_Position (0.0-100.0%)
     - Anti-windup: Freeze Integral when output clamped
     - Set ControlError, default Valve_Position to 0.0 on invalid input
   - Logs valve position changes and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
Valve_Position := 0.0;
Error := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor input *)
IF NOT IS_VALID_REAL(Level_PV) OR Level_PV < 0.0 OR Level_PV > 100.0 THEN
    ControlError := TRUE;
    Valve_Position := 0.0;
    Integral := 0.0; (* Reset integral on error *)
    IF LogCount < 50 AND (Level_PV <> LastLevel_PV OR ControlError <> LastControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:23:00'; (* Current date/time *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Level Input=', 
            CONCAT(TO_STRING(Level_PV), ' %'));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: PID control logic *)
    Error := Level_SP - Level_PV; (* Error in % *)
    Derivative := (Error - Prev_Error) / Sample_Time; (* %/s *)

    (* Anti-windup: Update integral only if output not clamped *)
    Valve_Position := Kp * Error + Ki * Integral + Kd * Derivative; (* Preliminary output *)
    IF Valve_Position >= Valve_Min AND Valve_Position <= Valve_Max THEN
        Integral := Integral + Error * Sample_Time; (* Update integral, %·s *)
    END_IF;

    (* Final output calculation *)
    Valve_Position := Kp * Error + Ki * Integral + Kd * Derivative;

    (* Step 4: Clamp valve position *)
    IF Valve_Position > Valve_Max THEN
        Valve_Position := Valve_Max; (* Clamp to 100.0% *)
    ELSIF Valve_Position < Valve_Min THEN
        Valve_Position := Valve_Min; (* Clamp to 0.0% *)
    END_IF;

    (* Step 5: Update previous error *)
    Prev_Error := Error;

    (* Step 6: Log large errors for diagnostics *)
    IF ABS(Error) > 20.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:23:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Error=', 
            CONCAT(TO_STRING(Error), ' %'));
    END_IF;
END_IF;

(* Step 7: Log valve position changes *)
IF Valve_Position <> LastValve_Position AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:23:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Valve Position: ', 
        CONCAT(TO_STRING(Valve_Position), '%'));
END_IF;
IF ControlError AND NOT LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:23:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND LastControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:23:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
LastLevel_PV := Level_PV;
LastValve_Position := Valve_Position;
LastControlError := ControlError;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: PID control for distillation column liquid level, maintains 60.0%.
   - Inputs:
     - Level_PV: REAL, measured liquid level (%).
   - Outputs:
     - Valve_Position: REAL, inlet valve signal (0.0-100.0%).
     - Error: REAL, control error (%).
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Error = 60.0 - Level_PV.
     - Integral += Error * 0.1, Derivative = (Error - Prev_Error) / 0.1.
     - Valve_Position = 1.5 * Error + 0.4 * Integral + 0.2 * Derivative.
     - Clamp Valve_Position (0.0-100.0%), anti-windup on Integral.
     - Validate Level_PV (0.0-100.0%).
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates input (finite, 0.0-100.0%).
     - Clamps output to safe range.
     - Anti-windup prevents integral overshoot.
     - Logs errors for diagnostics.
   - Usage:
     - Distillation column: Maintains 60.0% level, prevents flooding/drying.
     - Example: Level_PV=58.0 → Valve_Position adjusts to correct.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp uses current date/time; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
