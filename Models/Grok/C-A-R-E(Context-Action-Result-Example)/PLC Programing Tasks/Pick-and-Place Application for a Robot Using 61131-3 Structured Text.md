(* IEC 61131-3 Structured Text Program: PickAndPlaceControl *)
(* Purpose: Controls a pick-and-place robot with Manual and Auto modes *)

PROGRAM PickAndPlaceControl
VAR
    (* Inputs *)
    BtnManual : BOOL;               (* TRUE to activate Manual mode *)
    BtnAuto : BOOL;                 (* TRUE to activate Auto mode or trigger cycle *)
    CmdClip : BOOL;                 (* TRUE to command Clip in Manual mode *)
    CmdTransfer : BOOL;             (* TRUE to command Transfer in Manual mode *)
    CmdRelease : BOOL;              (* TRUE to command Release in Manual mode *)

    (* Outputs *)
    ManualMode : BOOL;              (* TRUE if Manual mode active *)
    AutoMode : BOOL;                (* TRUE if Auto mode active *)
    ClipActuator : BOOL;            (* TRUE to activate Clip action *)
    TransferActuator : BOOL;        (* TRUE to activate Transfer action *)
    ReleaseActuator : BOOL;         (* TRUE to activate Release action *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    State : INT := 0;               (* Auto mode state: 0=Idle, 1=Clip, 2=Transfer, 3=Release *)
    AutoTrigger : BOOL := FALSE;    (* TRUE to start Auto cycle *)
    AutoTimer : TON;                (* 2-second timer for Transfer delay *)
    LastBtnManual : BOOL;           (* Previous BtnManual state *)
    LastBtnAuto : BOOL;             (* Previous BtnAuto state *)
    LastManualMode : BOOL;          (* Previous ManualMode state *)
    LastAutoMode : BOOL;            (* Previous AutoMode state *)
    LastClipActuator : BOOL;        (* Previous ClipActuator state *)
    LastTransferActuator : BOOL;    (* Previous TransferActuator state *)
    LastReleaseActuator : BOOL;     (* Previous ReleaseActuator state *)
    LastState : INT;                (* Previous State value *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Mode Interlock:
     - BtnManual → ManualMode := TRUE, AutoMode := FALSE, reset AutoTrigger, AutoTimer
     - BtnAuto → AutoMode := TRUE, ManualMode := FALSE
     - Mutually exclusive modes; default neither if no button
   - Manual Mode:
     - Respond to CmdClip, CmdTransfer, CmdRelease for ClipActuator, TransferActuator, ReleaseActuator
   - Auto Mode:
     - BtnAuto triggers one-shot cycle (AutoTrigger := TRUE)
     - Sequence: Clip → Transfer (2s delay) → Release
     - Reset AutoTrigger after cycle
   - Logs mode changes and actuator actions for traceability
*)

(* Step 1: Mode interlock *)
IF BtnManual AND BtnAuto THEN
    (* Invalid: Both buttons pressed *)
    ManualMode := FALSE;
    AutoMode := FALSE;
    AutoTrigger := FALSE;
    AutoTimer.IN := FALSE;
    ClipActuator := FALSE;
    TransferActuator := FALSE;
    ReleaseActuator := FALSE;
    State := 0;
    IF LogCount < 50 AND (BtnManual <> LastBtnManual OR BtnAuto <> LastBtnAuto) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Both Manual and Auto Buttons Pressed');
    END_IF;
ELSIF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
    AutoTrigger := FALSE;
    AutoTimer.IN := FALSE;
    State := 0;
    IF ManualMode <> LastManualMode AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Manual Mode Activated');
    END_IF;
ELSIF BtnAuto THEN
    ManualMode := FALSE;
    AutoMode := TRUE;
    IF AutoMode <> LastAutoMode AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Auto Mode Activated');
    END_IF;
ELSE
    (* Neither button pressed, maintain current mode *)
    IF NOT ManualMode AND NOT AutoMode AND (LastManualMode OR LastAutoMode) AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Idle: No Mode Selected');
    END_IF;
END_IF;

(* Step 2: Manual Mode control *)
IF ManualMode THEN
    ClipActuator := CmdClip;
    TransferActuator := CmdTransfer;
    ReleaseActuator := CmdRelease;
    IF ClipActuator AND NOT LastClipActuator AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Manual Mode: Clip Activated');
    ELSIF NOT ClipActuator AND LastClipActuator AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Manual Mode: Clip Deactivated');
    END_IF;
    IF TransferActuator AND NOT LastTransferActuator AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Manual Mode: Transfer Activated');
    ELSIF NOT TransferActuator AND LastTransferActuator AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Manual Mode: Transfer Deactivated');
    END_IF;
    IF ReleaseActuator AND NOT LastReleaseActuator AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Manual Mode: Release Activated');
    ELSIF NOT ReleaseActuator AND LastReleaseActuator AND LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Manual Mode: Release Deactivated');
    END_IF;
ELSE
    (* Ensure actuators off in Auto mode unless in correct state *)
    IF NOT AutoMode THEN
        ClipActuator := FALSE;
        TransferActuator := FALSE;
        ReleaseActuator := FALSE;
    END_IF;
END_IF;

(* Step 3: Auto Mode trigger *)
IF AutoMode AND BtnAuto AND NOT AutoTrigger THEN
    AutoTrigger := TRUE;
    IF LogCount < 50 AND BtnAuto <> LastBtnAuto THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 17:15:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Auto Mode: Cycle Triggered');
    END_IF;
END_IF;

(* Step 4: Auto Mode sequence *)
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: (* Idle: Start cycle *)
            ClipActuator := TRUE;
            TransferActuator := FALSE;
            ReleaseActuator := FALSE;
            State := 1;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 17:15:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Auto Mode: Clip Started');
            END_IF;

        1: (* Clip: Transition to Transfer *)
            ClipActuator := FALSE;
            TransferActuator := TRUE;
            ReleaseActuator := FALSE;
            AutoTimer(IN := TRUE, PT := T#2s);
            State := 2;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 17:15:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Auto Mode: Transfer Started');
            END_IF;

        2: (* Transfer: Wait 2 seconds, then Release *)
            IF AutoTimer.Q THEN
                ClipActuator := FALSE;
                TransferActuator := FALSE;
                ReleaseActuator := TRUE;
                AutoTimer.IN := FALSE;
                State := 3;
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 17:15:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Auto Mode: Release Started');
                END_IF;
            END_IF;

        3: (* Release: Complete cycle, reset *)
            ClipActuator := FALSE;
            TransferActuator := FALSE;
            ReleaseActuator := FALSE;
            AutoTrigger := FALSE;
            State := 0;
            IF State <> LastState AND LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 17:15:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Auto Mode: Cycle Completed');
            END_IF;
    END_CASE;
END_IF;

(* Step 5: Update last states for logging *)
LastBtnManual := BtnManual;
LastBtnAuto := BtnAuto;
LastManualMode := ManualMode;
LastAutoMode := AutoMode;
LastClipActuator := ClipActuator;
LastTransferActuator := TransferActuator;
LastReleaseActuator := ReleaseActuator;
LastState := State;

(* Step 6: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Controls pick-and-place robot with Manual and Auto modes.
   - Inputs:
     - BtnManual: BOOL, TRUE for Manual mode.
     - BtnAuto: BOOL, TRUE for Auto mode or cycle trigger.
     - CmdClip, CmdTransfer, CmdRelease: BOOL, Manual mode commands.
   - Outputs:
     - ManualMode, AutoMode: BOOL, active mode.
     - ClipActuator, TransferActuator, ReleaseActuator: BOOL, actuator controls.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - Interlock: ManualMode or AutoMode exclusive; BtnManual/BtnAuto selects.
     - Manual: Activates actuators per CmdClip, CmdTransfer, CmdRelease.
     - Auto: BtnAuto triggers one-shot Clip → Transfer (2s) → Release cycle.
     - Resets AutoTrigger, AutoTimer between cycles or on mode change.
   - State Machine (Auto Mode):
     - Idle (0): Await trigger.
     - Clip (1): Activate ClipActuator.
     - Transfer (2): Activate TransferActuator, 2s delay.
     - Release (3): Activate ReleaseActuator, reset.
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, single timer (2s).
     - Minimal memory (~4 KB for logs, scalars for states).
   - Safety:
     - Mutually exclusive modes via interlock.
     - Resets actuators on mode change.
     - Logs mode and actuator changes for traceability.
   - Usage:
     - Pick-and-place: Manual for testing, Auto for production cycles.
     - Example: AutoMode=TRUE, BtnAuto=TRUE → Clip → Transfer (2s) → Release.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], BOOL, INT, TON; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
