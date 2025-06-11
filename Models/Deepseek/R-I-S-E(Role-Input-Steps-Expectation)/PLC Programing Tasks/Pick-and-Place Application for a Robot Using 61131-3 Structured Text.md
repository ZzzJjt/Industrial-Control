FUNCTION_BLOCK FB_RobotPickAndPlace
VAR_INPUT
    // Mode Selection
    BtnManual : BOOL; // Manual mode button (momentary)
    BtnAuto   : BOOL; // Auto mode button (momentary)

    // Manual Commands
    CmdClip     : BOOL; // Pick product from Conveyor A
    CmdTransfer : BOOL; // Move to Conveyor B
    CmdRelease  : BOOL; // Drop on Conveyor B
END_VAR

VAR_OUTPUT
    // Robot Actions
    ActClip     : BOOL := FALSE; // Output: Clip action active
    ActTransfer : BOOL := FALSE; // Output: Transfer in progress
    ActRelease  : BOOL := FALSE; // Output: Release action active

    // Mode Status
    ManualMode  : BOOL := FALSE;
    AutoMode    : BOOL := FALSE;
    AutoCycleActive : BOOL := FALSE;
END_VAR

VAR
    // Internal Logic
    AutoTrigger : BOOL := FALSE;
    State       : INT := 0;
    AutoTimer   : TON; // 2-second timer for transfer delay
END_VAR

// --- STEP 1: Mode Interlock Logic ---
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF;

// --- STEP 2: Manual Mode Control Logic ---
ActClip := FALSE;
ActTransfer := FALSE;
ActRelease := FALSE;

IF ManualMode THEN
    IF CmdClip THEN
        ActClip := TRUE;
    END_IF;

    IF CmdTransfer THEN
        ActTransfer := TRUE;
    END_IF;

    IF CmdRelease THEN
        ActRelease := TRUE;
    END_IF;
END_IF;

// --- STEP 3: Auto Mode Trigger Logic ---
IF BtnAuto AND NOT AutoMode THEN
    AutoTrigger := FALSE;
ELSIF BtnAuto AND AutoMode THEN
    AutoTrigger := TRUE;
END_IF;

// --- STEP 4: Auto Mode State Machine ---
IF AutoMode AND AutoTrigger THEN
    AutoCycleActive := TRUE;

    CASE State OF
        0: // CLIP Action
            ActClip := TRUE;
            State := 1;

        1: // TRANSFER Action with 2s Delay
            ActTransfer := TRUE;
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;

        2: // RELEASE Action
            ActRelease := TRUE;
            AutoTimer(IN := TRUE, PT := T#500ms); // Optional short hold
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                // Reset all actions and trigger
                ActClip := FALSE;
                ActTransfer := FALSE;
                ActRelease := FALSE;
                AutoTrigger := FALSE;
                AutoCycleActive := FALSE;
                State := 0;
            END_IF;
    END_CASE;
ELSE
    // Safely reset everything when not in use
    ActClip := FALSE;
    ActTransfer := FALSE;
    ActRelease := FALSE;
    AutoCycleActive := FALSE;
    State := 0;
    AutoTimer(IN := FALSE);
END_IF;

PROGRAM PLC_PRG
VAR
    RobotCtrl : FB_RobotPickAndPlace;

    // Simulated Inputs
    ManualBtn : BOOL := FALSE;
    AutoBtn   : BOOL := FALSE;

    ClipCmd     : BOOL := FALSE;
    TransferCmd : BOOL := FALSE;
    ReleaseCmd  : BOOL := FALSE;

    // Outputs / Actions
    ClipAction     : BOOL;
    TransferAction : BOOL;
    ReleaseAction  : BOOL;

    InManualMode   : BOOL;
    InAutoMode     : BOOL;
    CycleActive    : BOOL;
END_VAR

// Call the function block
RobotCtrl(
    BtnManual := ManualBtn,
    BtnAuto := AutoBtn,

    CmdClip := ClipCmd,
    CmdTransfer := TransferCmd,
    CmdRelease := ReleaseCmd,

    ActClip => ClipAction,
    ActTransfer => TransferAction,
    ActRelease => ReleaseAction,

    ManualMode => InManualMode,
    AutoMode => InAutoMode,
    AutoCycleActive => CycleActive
);
