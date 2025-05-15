PROGRAM PickAndPlaceControl
VAR_INPUT
    BtnManual : BOOL; // Button to activate Manual Mode
    BtnAuto : BOOL;   // Button to activate Auto Mode
    CmdClip : BOOL;   // Command to clip from Conveyor A (Manual Mode)
    CmdTransfer : BOOL; // Command to transfer to Conveyor B (Manual Mode)
    CmdRelease : BOOL; // Command to release onto Conveyor B (Manual Mode)
END_VAR

VAR_OUTPUT
    ManualMode : BOOL := FALSE; // Indicates if Manual Mode is active
    AutoMode : BOOL := FALSE;   // Indicates if Auto Mode is active
    ActuatorClip : BOOL;        // Actuates the clipping mechanism
    ActuatorTransfer : BOOL;    // Actuates the transfer mechanism
    ActuatorRelease : BOOL;     // Actuates the releasing mechanism
END_VAR

VAR
    AutoTrigger : BOOL := FALSE; // Trigger for starting an auto cycle
    State : INT := 0;            // State machine for Auto Mode
    AutoTimer : TON;             // Timer for delay in Auto Mode
    LastBtnManual : BOOL;        // Previous state of BtnManual
    LastBtnAuto : BOOL;          // Previous state of BtnAuto
END_VAR

// Interlock logic to switch modes
IF BtnManual AND NOT LastBtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
    AutoTrigger := FALSE; // Reset Auto mode trigger
    State := 0;           // Reset state machine
ELSIF BtnAuto AND NOT LastBtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
    AutoTrigger := FALSE; // Reset Auto mode trigger
    State := 0;           // Reset state machine
END_IF;

// Update previous button states
LastBtnManual := BtnManual;
LastBtnAuto := BtnAuto;

// Manual Mode control
IF ManualMode THEN
    ActuatorClip := CmdClip;
    ActuatorTransfer := CmdTransfer;
    ActuatorRelease := CmdRelease;
ELSE
    ActuatorClip := FALSE;
    ActuatorTransfer := FALSE;
    ActuatorRelease := FALSE;
END_IF;

// Auto Mode logic
IF AutoMode AND BtnAuto AND NOT AutoTrigger THEN
    AutoTrigger := TRUE;
END_IF;

IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: // Clip action
            ActuatorClip := TRUE;
            ActuatorTransfer := FALSE;
            ActuatorRelease := FALSE;
            State := 1;
        1: // Transfer delay
            ActuatorClip := FALSE;
            ActuatorTransfer := TRUE;
            ActuatorRelease := FALSE;
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;
        2: // Release action
            ActuatorClip := FALSE;
            ActuatorTransfer := FALSE;
            ActuatorRelease := TRUE;
            State := 3;
        3: // Complete cycle
            ActuatorClip := FALSE;
            ActuatorTransfer := FALSE;
            ActuatorRelease := FALSE;
            AutoTrigger := FALSE; // Reset Auto mode trigger
            State := 0;           // Reset state machine
    END_CASE;
END_IF;

// Inline comments explaining the logic:
// The program implements two interlocked modes: Manual and Auto.
// Manual Mode activates via a Manual button and responds to individual commands.
// Auto Mode activates via an Auto button and performs a predefined sequence with timing control.
// The program ensures that only one mode is active at a time and handles state transitions safely.



