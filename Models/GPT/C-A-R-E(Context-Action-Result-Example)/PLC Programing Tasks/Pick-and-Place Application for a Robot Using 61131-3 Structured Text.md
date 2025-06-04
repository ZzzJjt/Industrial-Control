PROGRAM PickAndPlaceControl
VAR
    // Mode selection buttons
    BtnManual    : BOOL;
    BtnAuto      : BOOL;

    // Control mode flags
    ManualMode   : BOOL := FALSE;
    AutoMode     : BOOL := FALSE;

    // Manual command buttons
    CmdClip      : BOOL;  // Clip from Conveyor A
    CmdTransfer  : BOOL;  // Transfer to Conveyor B
    CmdRelease   : BOOL;  // Release onto Conveyor B

    // Auto mode internal control
    AutoTrigger  : BOOL := FALSE;
    AutoCycleBtn : BOOL;  // Auto sequence trigger button

    // Actions to actuators
    DoClip       : BOOL := FALSE;
    DoTransfer   : BOOL := FALSE;
    DoRelease    : BOOL := FALSE;

    // Auto sequence state machine
    State        : INT := 0;
    AutoTimer    : TON;
END_VAR

// === Exclusive Mode Interlocking ===
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
    AutoTrigger := FALSE;
    State := 0;
END_IF;

IF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF;

// === Manual Mode Logic ===
IF ManualMode THEN
    IF CmdClip THEN
        DoClip := TRUE;
    ELSE
        DoClip := FALSE;
    END_IF;

    IF CmdTransfer THEN
        DoTransfer := TRUE;
    ELSE
        DoTransfer := FALSE;
    END_IF;

    IF CmdRelease THEN
        DoRelease := TRUE;
    ELSE
        DoRelease := FALSE;
    END_IF;
END_IF;

// === Auto Mode Trigger ===
IF AutoMode AND AutoCycleBtn THEN
    AutoTrigger := TRUE;
END_IF;

// === Auto Mode Logic ===
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: // Start: Clip
            DoClip := TRUE;
            DoTransfer := FALSE;
            DoRelease := FALSE;
            State := 1;

        1: // Transfer after Clip
            DoClip := FALSE;
            DoTransfer := TRUE;
            AutoTimer(IN := TRUE, PT := T#2s); // 2-second delay
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;

        2: // Release
            DoTransfer := FALSE;
            DoRelease := TRUE;
            State := 3;

        3: // Reset sequence
            DoRelease := FALSE;
            AutoTrigger := FALSE;
            State := 0;
    END_CASE;
ELSE
    // Reset outputs and timer if AutoMode is not running
    DoClip := FALSE;
    DoTransfer := FALSE;
    DoRelease := FALSE;
    AutoTimer(IN := FALSE);
END_IF;
