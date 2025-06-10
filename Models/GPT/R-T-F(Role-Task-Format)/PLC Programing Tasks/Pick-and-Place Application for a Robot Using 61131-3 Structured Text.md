PROGRAM PickAndPlaceControl
VAR
    // Mode selection buttons
    BtnManual     : BOOL;
    BtnAuto       : BOOL;

    // Mode status flags
    ManualMode    : BOOL := FALSE;
    AutoMode      : BOOL := FALSE;

    // Manual command buttons
    CmdClip       : BOOL;
    CmdTransfer   : BOOL;
    CmdRelease    : BOOL;

    // Control outputs
    ClipActuator     : BOOL;
    TransferActuator : BOOL;
    ReleaseActuator  : BOOL;

    // Auto mode control
    AutoTrigger   : BOOL := FALSE;
    State         : INT := 0;

    // Timer for 2-second delay
    AutoTimer     : TON;
END_VAR

// --- Mode Interlock: Only one mode active at a time ---
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF

// --- Manual Mode Logic ---
IF ManualMode THEN
    // Clip command
    IF CmdClip THEN
        ClipActuator := TRUE;
    ELSE
        ClipActuator := FALSE;
    END_IF;

    // Transfer command
    IF CmdTransfer THEN
        TransferActuator := TRUE;
    ELSE
        TransferActuator := FALSE;
    END_IF;

    // Release command
    IF CmdRelease THEN
        ReleaseActuator := TRUE;
    ELSE
        ReleaseActuator := FALSE;
    END_IF;
END_IF

// --- Auto Mode Trigger: Execute one cycle per button press ---
IF AutoMode AND BtnAuto THEN
    AutoTrigger := TRUE;
END_IF

// --- Auto Mode Sequence Logic ---
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: // Step 1: Clip product
            ClipActuator := TRUE;
            TransferActuator := FALSE;
            ReleaseActuator := FALSE;
            State := 1;

        1: // Step 2: Transfer after 2-second delay
            ClipActuator := FALSE;
            TransferActuator := TRUE;
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE); // Stop the timer
                State := 2;
            END_IF;

        2: // Step 3: Release product
            TransferActuator := FALSE;
            ReleaseActuator := TRUE;
            State := 3;

        3: // Step 4: Finish and reset
            ReleaseActuator := FALSE;
            AutoTrigger := FALSE;  // Allow new trigger next cycle
            State := 0;
    END_CASE;
ELSE
    // Reset all outputs when auto not active
    IF NOT ManualMode THEN
        ClipActuator := FALSE;
        TransferActuator := FALSE;
        ReleaseActuator := FALSE;
        AutoTimer(IN := FALSE);
    END_IF;
END_IF
