VAR
    // Mode control
    ManualMode, AutoMode : BOOL := FALSE;
    BtnManual, BtnAuto : BOOL;         // Mode selection buttons

    // Manual commands
    CmdClip, CmdTransfer, CmdRelease : BOOL;

    // Auto sequence
    AutoTrigger : BOOL := FALSE;       // Rising-edge trigger
    AutoButtonPrev : BOOL := FALSE;    // For edge detection
    State : INT := 0;

    // Timing for transfer step
    AutoTimer : TON;

    // Outputs to actuator
    DO_Clip : BOOL := FALSE;
    DO_Transfer : BOOL := FALSE;
    DO_Release : BOOL := FALSE;
END_VAR

// === Mode Interlocking ===
IF BtnManual THEN
    ManualMode := TRUE;
    AutoMode := FALSE;
ELSIF BtnAuto THEN
    AutoMode := TRUE;
    ManualMode := FALSE;
END_IF

// === Manual Mode Control ===
IF ManualMode THEN
    // Directly activate actuators
    DO_Clip := CmdClip;
    DO_Transfer := CmdTransfer;
    DO_Release := CmdRelease;
ELSE
    // Disable all manual outputs in Auto mode
    DO_Clip := FALSE;
    DO_Transfer := FALSE;
    DO_Release := FALSE;
END_IF

// === Auto Mode One-Shot Trigger ===
IF AutoMode AND BtnAuto AND NOT AutoButtonPrev THEN
    AutoTrigger := TRUE;
END_IF
AutoButtonPrev := BtnAuto;

// === Auto Mode State Machine ===
IF AutoMode AND AutoTrigger THEN
    CASE State OF
        0: // Clip
            DO_Clip := TRUE;
            DO_Transfer := FALSE;
            DO_Release := FALSE;
            State := 1;

        1: // Transfer with delay
            DO_Clip := FALSE;
            DO_Transfer := TRUE;
            DO_Release := FALSE;

            AutoTimer(IN := TRUE, PT := T#2s);
            IF AutoTimer.Q THEN
                AutoTimer(IN := FALSE);
                State := 2;
            END_IF;

        2: // Release
            DO_Clip := FALSE;
            DO_Transfer := FALSE;
            DO_Release := TRUE;
            AutoTrigger := FALSE;
            State := 0;
    END_CASE;
ELSE
    // Ensure outputs off outside active state
    IF NOT ManualMode THEN
        DO_Clip := FALSE;
        DO_Transfer := FALSE;
        DO_Release := FALSE;
    END_IF;
END_IF
