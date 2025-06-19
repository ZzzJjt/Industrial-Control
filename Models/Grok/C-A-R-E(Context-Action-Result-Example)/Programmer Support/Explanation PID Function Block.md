(* IEC 61131-3 Structured Text Program: VesselPressureCascadeControl *)
(* Purpose: Implements cascade control for vessel pressure regulation *)

PROGRAM VesselPressureCascadeControl
VAR
    (* Inputs *)
    PV1 : REAL;                     (* Measured vessel pressure, bar *)
    PV2 : REAL;                     (* Measured flow rate, L/min *)

    (* Outputs *)
    OP2 : REAL;                     (* Valve position, 0.0-100.0% *)
    SP2 : REAL;                     (* Flow setpoint for secondary loop, L/min *)
    e1 : REAL;                      (* Primary loop error, bar *)
    e2 : REAL;                      (* Secondary loop error, L/min *)
    ControlError : BOOL := FALSE;   (* TRUE if input/control fault *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    SP1 : REAL := 5.0;              (* Pressure setpoint, bar *)
    Kp1 : REAL := 2.0;              (* Primary loop proportional gain, L/min/bar *)
    Ki1 : REAL := 0.5;              (* Primary loop integral gain, L/min/(bar·s) *)
    Kd1 : REAL := 0.2;              (* Primary loop derivative gain, L/min/(bar/s) *)
    Kp2 : REAL := 3.0;              (* Secondary loop proportional gain, % valve/(L/min) *)
    Ki2 : REAL := 0.8;              (* Secondary loop integral gain, % valve/((L/min)·s) *)
    Kd2 : REAL := 0.3;              (* Secondary loop derivative gain, % valve/((L/min)/s) *)
    dt : REAL := 0.1;               (* Sample time, 100 ms *)
    e1_sum : REAL := 0.0;           (* Primary loop integral term, bar·s *)
    e1_prev : REAL := 0.0;          (* Previous primary loop error, bar *)
    e1_diff : REAL;                 (* Primary loop derivative term, bar/s *)
    OP1 : REAL;                     (* Primary loop output, L/min *)
    e2_sum : REAL := 0.0;           (* Secondary loop integral term, (L/min)·s *)
    e2_prev : REAL := 0.0;          (* Previous secondary loop error, L/min *)
    e2_diff : REAL;                 (* Secondary loop derivative term, (L/min)/s *)
    Last_PV1 : REAL;                (* Previous PV1 value *)
    Last_PV2 : REAL;                (* Previous PV2 value *)
    Last_OP2 : REAL;                (* Previous OP2 value *)
    Last_SP2 : REAL;                (* Previous SP2 value *)
    Last_ControlError : BOOL;       (* Previous ControlError state *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Primary Loop (Pressure):
     - e1 = SP1 - PV1
     - e1_sum += e1 * 0.1, e1_diff = (e1 - e1_prev) / 0.1
     - OP1 = Kp1 * e1 + Ki1 * e1_sum + Kd1 * e1_diff
     - Clamp OP1 (0.0-100.0 L/min), anti-windup on e1_sum
   - Secondary Loop (Flow):
     - SP2 = OP1
     - e2 = SP2 - PV2
     - e2_sum += e2 * 0.1, e2_diff = (e2 - e2_prev) / 0.1
     - OP2 = Kp2 * e2 + Ki2 * e2_sum + Kd2 * e2_diff
     - Clamp OP2 (0.0-100.0%), anti-windup on e2_sum
   - Safety:
     - Validate PV1 (0.0-10.0 bar), PV2 (0.0-100.0 L/min)
     - Set ControlError, default OP2 to 0.0 on invalid input
   - Logs valve position, setpoints, and errors for diagnostics
*)

(* Step 1: Initialize outputs *)
OP2 := 0.0;
SP2 := 0.0;
e1 := 0.0;
e2 := 0.0;
ControlError := FALSE;

(* Step 2: Validate sensor inputs *)
IF NOT IS_VALID_REAL(PV1) OR NOT IS_VALID_REAL(PV2) OR
   PV1 < 0.0 OR PV1 > 10.0 OR PV2 < 0.0 OR PV2 > 100.0 THEN
    ControlError := TRUE;
    OP2 := 0.0;
    SP2 := 0.0;
    e1_sum := 0.0; (* Reset primary integral *)
    e2_sum := 0.0; (* Reset secondary integral *)
    IF LogCount < 50 AND (PV1 <> Last_PV1 OR PV2 <> Last_PV2 OR ControlError <> Last_ControlError) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:35:00'; (* Current date/time *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Input - Pressure=', 
            CONCAT(TO_STRING(PV1), CONCAT(' bar, Flow=', 
            CONCAT(TO_STRING(PV2), ' L/min'))));
    END_IF;
ELSE
    ControlError := FALSE;

    (* Step 3: Primary loop - Pressure control *)
    e1 := SP1 - PV1; (* Error in bar *)
    e1_diff := (e1 - e1_prev) / dt; (* bar/s *)

    (* Anti-windup: Update integral only if output not clamped *)
    OP1 := Kp1 * e1 + Ki1 * e1_sum + Kd1 * e1_diff; (* Preliminary output *)
    IF OP1 >= 0.0 AND OP1 <= 100.0 THEN
        e1_sum := e1_sum + e1 * dt; (* Update integral, bar·s *)
    END_IF;

    (* Final primary output calculation *)
    OP1 := Kp1 * e1 + Ki1 * e1_sum + Kd1 * e1_diff;

    (* Clamp OP1 to flow setpoint limits *)
    IF OP1 > 100.0 THEN
        OP1 := 100.0; (* Clamp to 100.0 L/min *)
    ELSIF OP1 < 0.0 THEN
        OP1 := 0.0; (* Clamp to 0.0 L/min *)
    END_IF;

    (* Step 4: Secondary loop - Flow control *)
    SP2 := OP1; (* Flow setpoint from primary loop *)
    e2 := SP2 - PV2; (* Error in L/min *)
    e2_diff := (e2 - e2_prev) / dt; (* (L/min)/s *)

    (* Anti-windup: Update integral only if output not clamped *)
    OP2 := Kp2 * e2 + Ki2 * e2_sum + Kd2 * e2_diff; (* Preliminary output *)
    IF OP2 >= 0.0 AND OP2 <= 100.0 THEN
        e2_sum := e2_sum + e2 * dt; (* Update integral, (L/min)·s *)
    END_IF;

    (* Final secondary output calculation *)
    OP2 := Kp2 * e2 + Ki2 * e2_sum + Kd2 * e2_diff;

    (* Clamp OP2 to valve position limits *)
    IF OP2 > 100.0 THEN
        OP2 := 100.0; (* Clamp to 100.0% *)
    ELSIF OP2 < 0.0 THEN
        OP2 := 0.0; (* Clamp to 0.0% *)
    END_IF;

    (* Step 5: Update previous errors *)
    e1_prev := e1;
    e2_prev := e2;

    (* Step 6: Log large errors for diagnostics *)
    IF ABS(e1) > 2.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19ought to be 12:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Pressure Error=', 
            CONCAT(TO_STRING(e1), ' bar'));
    END_IF;
    IF ABS(e2) > 20.0 AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Warning: Large Flow Error=', 
            CONCAT(TO_STRING(e2), ' L/min'));
    END_IF;
END_IF;

(* Step 7: Log setpoint and valve position changes *)
IF SP2 <> Last_SP2 AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:35:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Flow Setpoint: ', 
        CONCAT(TO_STRING(SP2), ' L/min'));
END_IF;
IF OP2 <> Last_OP2 AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:35:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Valve Position: ', 
        CONCAT(TO_STRING(OP2), '%'));
END_IF;
IF ControlError AND NOT Last_ControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Active');
    END_IF;
ELSIF NOT ControlError AND Last_ControlError THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Control Error Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
Last_PV1 := PV1;
Last_PV2 := PV2;
Last_OP2 := OP2;
Last_SP2 := SP2;
Last_ControlError := ControlError;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Cascade control for vessel pressure, primary loop sets flow setpoint, secondary loop adjusts valve.
   - Inputs:
     - PV1: REAL, measured pressure (bar).
     - PV2: REAL, measured flow rate (L/min).
   - Outputs:
     - OP2: REAL, valve position (0.0-100.0%).
     - SP2: REAL, flow setpoint (L/min).
     - e1, e2: REAL, control errors (bar, L/min).
     - ControlError: BOOL, TRUE for input/control faults.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Primary: e1 = 5.0 - PV1, OP1 = 2.0 * e1 + 0.5 * e1_sum + 0.2 * e1_diff, clamp OP1 (0.0-100.0 L/min).
     - Secondary: SP2 = OP1, e2 = SP2 - PV2, OP2 = 3.0 * e2 + 0.8 * e2_sum + 0.3 * e2_diff, clamp OP2 (0.0-100.0%).
     - Anti-windup on e1_sum, e2_sum.
     - Validate PV1 (0.0-10.0 bar), PV2 (0.0-100.0 L/min).
   - Optimization:
     - Simple logic (~100 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for inputs).
   - Safety:
     - Validates inputs (finite, ranges).
     - Clamps outputs to safe ranges.
     - Anti-windup prevents integral overshoot.
     - Logs errors for diagnostics.
   - Usage:
     - Vessel pressure: Maintains 5.0 bar via flow control.
     - Example: PV1=4.8, PV2=50.0 → SP2 and OP2 adjust to correct.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp uses current date/time; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
