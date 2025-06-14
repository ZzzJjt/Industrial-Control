VAR
    // Mode control inputs
    BtnManual       : BOOL;
    BtnAuto         : BOOL;

    // Manual command inputs
    CmdClip         : BOOL;
    CmdTransfer     : BOOL;
    CmdRelease      : BOOL;

    // Mode flags
    ManualMode      : BOOL := FALSE;
    AutoMode        : BOOL := FALSE;

    // Auto mode trigger and state
    AutoTrigger     : BOOL := FALSE;
    State           : INT := 0;

    // Timer for Auto Mode delay
    AutoTimer       : TON;

    // Outputs to actuators
    ActClip         : BOOL := FALSE;
    ActTransfer     : BOOL := FALSE;
    ActRelease      : BOOL := FALSE;
END_VAR

//------------------------//
// Interlock Mode Logic   //
//------------------------//
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF

//-----------------------------------//
// Manual Mode – Direct Command Logic //
//-----------------------------------//
IF ManualMode THEN
    IF CmdClip THEN
        ActClip := TRUE;
    ELSE
        ActClip := FALSE;
    END_IF;

    IF CmdTransfer THEN
        ActTransfer := TRUE;
    ELSE
        ActTransfer := FALSE;
    END_IF;

    IF CmdRelease THEN
        ActRelease := TRUE;
    ELSE
        ActRelease := FALSE;
    END_IF;
END_IF

//-------------------------------------//
// Auto Mode – Full Sequence Logic     //
//-------------------------------------//

// Trigger only once per BtnAuto press (rising edge logic can be added externally if needed)
IF AutoMode AND BtnAuto THEN
    AutoTrigger := TRUE;
END_IF

IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: // Start – Clip
            ActClip := TRUE;
            ActTransfer := FALSE;
            ActRelease := FALSE;
            State := 1;
        
        1: // Start Timer after Clip
            ActClip := FALSE;
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;

        2: // Release
            ActRelease := TRUE;
            ActTransfer := FALSE;
            State := 3;

        3: // Reset
            ActRelease := FALSE;
            AutoTrigger := FALSE;
            State := 0;
    END_CASE;
ELSE
    // Default deactivation if not in auto sequence
    IF NOT ManualMode THEN
        ActClip := FALSE;
        ActTransfer := FALSE;
        ActRelease := FALSE;
    END_IF
END_IF
