FUNCTION_BLOCK PickAndPlaceRobot
VAR_INPUT
    BtnManual     : BOOL; // Manual mode selection
    BtnAuto       : BOOL; // Auto mode selection (also triggers auto cycle)
    CmdClip       : BOOL; // Manual command: pick from Conveyor A
    CmdTransfer   : BOOL; // Manual command: move to Conveyor B
    CmdRelease    : BOOL; // Manual command: release on Conveyor B
END_VAR

VAR_OUTPUT
    DoClip        : BOOL; // Actuate clip action
    DoTransfer    : BOOL; // Actuate transfer motion
    DoRelease     : BOOL; // Actuate release motion
END_VAR

VAR
    ManualMode    : BOOL := FALSE;
    AutoMode      : BOOL := FALSE;
    AutoTrigger   : BOOL := FALSE;
    PrevBtnAuto   : BOOL := FALSE;
    State         : INT := 0; // 0=Clip, 1=Wait, 2=Release
    AutoTimer     : TON; // Timer for 2s delay
END_VAR

// --- Mode Selection Interlock ---
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF

// --- Manual Mode Logic ---
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
ELSE
    // Clear outputs if not in manual
    DoClip := FALSE;
    DoTransfer := FALSE;
    DoRelease := FALSE;
END_IF

// --- Auto Mode Rising Edge Detection ---
IF AutoMode AND (BtnAuto AND NOT PrevBtnAuto) THEN
    AutoTrigger := TRUE;
END_IF
PrevBtnAuto := BtnAuto;

// --- Auto Mode Logic with State Machine ---
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: // Step 1: Clip
            DoClip := TRUE;
            DoTransfer := FALSE;
            DoRelease := FALSE;
            State := 1;
        1: // Step 2: Delay and Transfer
            DoClip := FALSE;
            DoTransfer := TRUE;
            DoRelease := FALSE;
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;
        2: // Step 3: Release
            DoClip := FALSE;
            DoTransfer := FALSE;
            DoRelease := TRUE;
            AutoTrigger := FALSE;
            State := 0; // Reset for next cycle
    END_CASE;
ELSE
    // Ensure outputs are off when Auto is idle
    IF NOT AutoMode THEN
        DoClip := FALSE;
        DoTransfer := FALSE;
        DoRelease := FALSE;
        AutoTimer(IN := FALSE);
        State := 0;
        AutoTrigger := FALSE;
    END_IF
END_IF
