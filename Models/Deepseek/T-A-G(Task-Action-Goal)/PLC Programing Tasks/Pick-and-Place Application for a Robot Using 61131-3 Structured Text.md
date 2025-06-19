PROGRAM PickAndPlaceControl
VAR_INPUT
    BtnManual : BOOL; // Button to activate Manual Mode
    BtnAuto : BOOL;   // Button to activate Auto Mode
    CmdClip : BOOL;   // Command to clip a product in Manual Mode
    CmdTransfer : BOOL; // Command to transfer a product in Manual Mode
    CmdRelease : BOOL; // Command to release a product in Manual Mode
END_VAR

VAR_OUTPUT
    ClipActuator : BOOL; // Actuator to clip the product
    TransferActuator : BOOL; // Actuator to transfer the product
    ReleaseActuator : BOOL; // Actuator to release the product
END_VAR

VAR
    ManualMode : BOOL := FALSE; // Flag indicating Manual Mode
    AutoMode : BOOL := FALSE; // Flag indicating Auto Mode
    AutoTrigger : BOOL := FALSE; // Trigger for starting the auto sequence
    State : INT := 0; // State variable for the state machine
    AutoTimer : TON; // Timer for the 2-second delay in Auto Mode
END_VAR

// Interlock modes
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF;

// Manual mode logic
IF ManualMode THEN
    IF CmdClip THEN
        ClipActuator := TRUE;
        TransferActuator := FALSE;
        ReleaseActuator := FALSE;
    ELSIF CmdTransfer THEN
        ClipActuator := FALSE;
        TransferActuator := TRUE;
        ReleaseActuator := FALSE;
    ELSIF CmdRelease THEN
        ClipActuator := FALSE;
        TransferActuator := FALSE;
        ReleaseActuator := TRUE;
    ELSE
        ClipActuator := FALSE;
        TransferActuator := FALSE;
        ReleaseActuator := FALSE;
    END_IF;
ELSE
    ClipActuator := FALSE;
    TransferActuator := FALSE;
    ReleaseActuator := FALSE;
END_IF;

// Auto mode trigger
IF AutoMode AND BtnAuto THEN
    AutoTrigger := TRUE;
END_IF;

// Auto sequence logic
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: // Initial state, wait for trigger
            State := 1;
        1: // Clip action
            ClipActuator := TRUE;
            TransferActuator := FALSE;
            ReleaseActuator := FALSE;
            State := 2;
        2: // Wait for 2 seconds
            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 3;
            END_IF;
        3: // Release action
            ClipActuator := FALSE;
            TransferActuator := FALSE;
            ReleaseActuator := TRUE;
            State := 4;
        4: // Reset sequence and clear triggers
            AutoTrigger := FALSE;
            ClipActuator := FALSE;
            TransferActuator := FALSE;
            ReleaseActuator := FALSE;
            State := 0;
    END_CASE;
END_IF;

// Additional comments for clarity
// - Interlock modes: Ensure only one mode (Manual or Auto) is active at a time.
// - Manual mode logic: Respond to individual commands (CmdClip, CmdTransfer, CmdRelease).
// - Auto mode trigger: Start the auto sequence when BtnAuto is pressed in AutoMode.
// - Auto sequence logic: Use a state machine and a TON timer for the 2-second delay.
//   - State 0: Initial state, wait for trigger.
//   - State 1: Perform the clip action.
//   - State 2: Wait for 2 seconds using the AutoTimer.
//   - State 3: Perform the release action.
//   - State 4: Reset the sequence and clear triggers for the next cycle.



