(* IEC 61131-3 Structured Text: CarWashControl Function Block *)
(* Purpose: Controls an automatic car wash with safety interlocks *)

FUNCTION_BLOCK CarWashControl
VAR_INPUT
    CarPresentSensor : BOOL;        (* TRUE if vehicle detected in wash bay *)
    HumanDetectedSensor : BOOL;     (* TRUE if person detected in wash area *)
END_VAR
VAR_OUTPUT
    WashActive : BOOL;              (* TRUE to activate wash cycle *)
    AlarmActive : BOOL;             (* TRUE to trigger safety alarm *)
    SafeToRun : BOOL;               (* TRUE if system is safe to start wash *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Idle, 1=Running, 2=Stopped, 3=Alarm *)
    LastCarPresent : BOOL;          (* Previous CarPresentSensor state *)
    LastHumanDetected : BOOL;       (* Previous HumanDetectedSensor state *)
    LastState : INT;                (* Previous state for logging *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Start wash if: Car present, no human detected, SafeToRun = TRUE
   - Stop wash, activate alarm, block operation if human detected
   - Resume SafeToRun only when no human and wash is off
   - State machine ensures clear transitions and interlocks
*)

(* Step 1: Initialize outputs *)
WashActive := FALSE;
AlarmActive := FALSE;
SafeToRun := TRUE; (* Initial state assumes safe unless human detected *)

(* Step 2: State machine execution *)
CASE State OF
    0: (* Idle: No car, wash off, safe to run *)
        WashActive := FALSE;
        AlarmActive := FALSE;
        SafeToRun := NOT HumanDetectedSensor;
        IF CarPresentSensor AND NOT HumanDetectedSensor AND SafeToRun THEN
            State := 1; (* Transition to Running *)
            WashActive := TRUE;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 11:25:00'; (* Replace with system clock *)
                DiagLog[LogCount] := CONCAT(Timestamp, ' Wash Started: Car Detected');
            END_IF;
        END_IF;

    1: (* Running: Car present, wash active *)
        WashActive := TRUE;
        AlarmActive := FALSE;
        SafeToRun := NOT HumanDetectedSensor;
        IF HumanDetectedSensor THEN
            State := 3; (* Transition to Alarm *)
            WashActive := FALSE;
            AlarmActive := TRUE;
            SafeToRun := FALSE;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 11:25:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Human Detected: Alarm Activated');
            END_IF;
        ELSIF NOT CarPresentSensor THEN
            State := 0; (* Transition to Idle *)
            WashActive := FALSE;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 11:25:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Wash Stopped: Car Exited');
            END_IF;
        END_IF;

    2: (* Stopped: Car present, wash off, safe to run if no human *)
        WashActive := FALSE;
        AlarmActive := FALSE;
        SafeToRun := NOT HumanDetectedSensor;
        IF HumanDetectedSensor THEN
            State := 3; (* Transition to Alarm *)
            AlarmActive := TRUE;
            SafeToRun := FALSE;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 11:25:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Human Detected: Alarm Activated');
            END_IF;
        ELSIF CarPresentSensor AND NOT HumanDetectedSensor AND SafeToRun THEN
            State := 1; (* Transition to Running *)
            WashActive := TRUE;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 11:25:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Wash Started: Car Detected');
            END_IF;
        ELSIF NOT CarPresentSensor THEN
            State := 0; (* Transition to Idle *)
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 11:25:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Idle: No Car Present');
            END_IF;
        END_IF;

    3: (* Alarm: Human detected, wash off, not safe to run *)
        WashActive := FALSE;
        AlarmActive := TRUE;
        SafeToRun := FALSE;
        IF NOT HumanDetectedSensor AND NOT WashActive THEN
            State := 2; (* Transition to Stopped *)
            AlarmActive := FALSE;
            SafeToRun := TRUE;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 11:25:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Area Clear: Alarm Cleared');
            END_IF;
        END_IF;
END_CASE;

(* Step 3: Update last states for logging *)
LastCarPresent := CarPresentSensor;
LastHumanDetected := HumanDetectedSensor;
LastState := State;

(* Step 4: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Controls car wash with safety interlocks for human presence.
   - Inputs:
     - CarPresentSensor: BOOL, TRUE if vehicle detected.
     - HumanDetectedSensor: BOOL, TRUE if person detected.
   - Outputs:
     - WashActive: BOOL, TRUE to activate wash cycle.
     - AlarmActive: BOOL, TRUE to trigger alarm.
     - SafeToRun: BOOL, TRUE if safe to start wash.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Start wash: Car present, no human, SafeToRun.
     - Stop wash: Human detected, activate alarm, block operation.
     - Resume: No human, wash off, set SafeToRun.
   - State Machine:
     - Idle (0): No car, wash off, safe if no human.
     - Running (1): Car present, wash on, no human.
     - Stopped (2): Car present, wash off, safe if no human.
     - Alarm (3): Human detected, wash off, alarm on.
   - Optimization:
     - Simple logic (~20 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, deterministic execution.
     - Minimal memory (~4 KB for logs, scalars for states).
   - Safety:
     - Prioritizes human safety: Immediate stop on human detection.
     - Interlocks prevent unsafe operation (SafeToRun).
     - Logs state changes for traceability.
   - Usage:
     - Car wash: Starts wash with car, stops on human detection, resumes when clear.
     - Example: CarPresentSensor=TRUE, HumanDetectedSensor=FALSE → WashActive=TRUE.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], BOOL, INT; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
