PROGRAM PLC_PRG
TITLE 'Robotic Pick-and-Place Control – Manual & Auto Mode'

(*
    Description:
    Controls a robot performing pick-and-place operations with two operating modes:
    
    - Manual Mode: Step-by-step control via individual commands
    - Auto Mode: Fully automatic sequence triggered by one button
    
    Features:
    - Mutual exclusion between modes
    - Safe interlocks to prevent command conflicts
    - State-based auto sequencing with delay
    - Timer-controlled transfer phase
    
    Safety:
    - Only one mode active at any time
    - Auto sequence only runs when mode is active
*)

VAR
    // Inputs: Mode Selection
    BtnManual : BOOL := FALSE;     // Operator selects Manual Mode
    BtnAuto   : BOOL := FALSE;     // Operator selects Auto Mode

    // Inputs: Manual Commands
    CmdClip     : BOOL := FALSE;   // Pick from conveyor A
    CmdTransfer : BOOL := FALSE;   // Move to conveyor B
    CmdRelease  : BOOL := FALSE;   // Drop on conveyor B

    // Internal Flags
    ManualMode  : BOOL := TRUE;    // Default to Manual Mode
    AutoMode    : BOOL := FALSE;
    AutoTrigger : BOOL := FALSE;
    State       : INT := 0;        // State machine for Auto Sequence
    AutoTimer   : TON;             // Delay timer for Transfer phase

    // Outputs: Simulated Robot Actions
    ClipAct     : BOOL := FALSE;   // Physical clip actuation
    TransferAct : BOOL := FALSE;   // Transfer movement
    ReleaseAct  : BOOL := FALSE;  // Release mechanism
END_VAR

// === MAIN LOGIC ===

// --- Mode Interlock Logic ---
// Only one mode can be active at a time
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF;

// --- Manual Mode Operations ---
IF ManualMode THEN
    IF CmdClip THEN
        ClipAct := TRUE;
    ELSE
        ClipAct := FALSE;
    END_IF;

    IF CmdTransfer THEN
        TransferAct := TRUE;
    ELSE
        TransferAct := FALSE;
    END_IF;

    IF CmdRelease THEN
        ReleaseAct := TRUE;
    ELSE
        ReleaseAct := FALSE;
    END_IF;
END_IF;

// --- Auto Mode Trigger ---
// Auto trigger is set once per button press
IF AutoMode AND BtnAuto THEN
    AutoTrigger := TRUE;
END_IF;

// --- Auto Mode Sequence (State Machine) ---
IF AutoMode AND AutoTrigger THEN

    CASE State OF
        0: // STEP 0: Clip action
            ClipAct := TRUE;
            TransferAct := FALSE;
            ReleaseAct := FALSE;
            State := 1;

        1: // STEP 1: Wait 2 seconds before transfer
            ClipAct := FALSE;
            TransferAct := TRUE;
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;

        2: // STEP 2: Release and reset
            TransferAct := FALSE;
            ReleaseAct := TRUE;
            AutoTrigger := FALSE;
            State := 0;

    END_CASE;

ELSE
    // Reset all actions if not in Auto Mode or after sequence ends
    ClipAct := FALSE;
    TransferAct := FALSE;
    ReleaseAct := FALSE;
END_IF;

END_PROGRAM
