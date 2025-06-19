PROGRAM ESM_Controller
VAR
    // Enumerated EtherCAT states
    STATE_INIT    : INT := 0;
    STATE_PREOP   : INT := 1;
    STATE_SAFEOP  : INT := 2;
    STATE_OP      : INT := 3;

    // Current and target state
    CurrentState  : INT := 0;
    TargetState   : INT := 1; // PREOP initially
    StateStep     : INT := 0;

    // Timer for stabilization delay
    DelayTimer    : TON;
    TimerStarted  : BOOL := FALSE;

    // State command and feedback
    CommandIssued : BOOL := FALSE;
    EtherCAT_FB   : INT;      // Read current device state from ESM
    EtherCAT_CMD  : INT;      // Write new device state to ESM

    // Error handling
    ErrorDetected : BOOL := FALSE;
    ErrorMessage  : STRING[100];
END_VAR

// Read current EtherCAT state (this would be via a function or device read)
CurrentState := EtherCAT_FB; // Simulated value; replace with real device input

// Main transition logic
CASE StateStep OF
    0: // INIT → PREOP
        IF NOT TimerStarted THEN
            EtherCAT_CMD := STATE_PREOP;
            CommandIssued := TRUE;
            DelayTimer(IN := TRUE, PT := T#5s);
            TimerStarted := TRUE;
        ELSIF DelayTimer.Q THEN
            IF CurrentState = STATE_PREOP THEN
                StateStep := 1;
                TimerStarted := FALSE;
                DelayTimer(IN := FALSE);
            ELSE
                ErrorDetected := TRUE;
                ErrorMessage := 'Failed to enter PREOP';
            END_IF
        END_IF

    1: // PREOP → SAFEOP
        IF NOT TimerStarted THEN
            EtherCAT_CMD := STATE_SAFEOP;
            CommandIssued := TRUE;
            DelayTimer(IN := TRUE, PT := T#5s);
            TimerStarted := TRUE;
        ELSIF DelayTimer.Q THEN
            IF CurrentState = STATE_SAFEOP THEN
                StateStep := 2;
                TimerStarted := FALSE;
                DelayTimer(IN := FALSE);
            ELSE
                ErrorDetected := TRUE;
                ErrorMessage := 'Failed to enter SAFEOP';
            END_IF
        END_IF

    2: // SAFEOP → OP
        IF NOT TimerStarted THEN
            EtherCAT_CMD := STATE_OP;
            CommandIssued := TRUE;
            DelayTimer(IN := TRUE, PT := T#5s);
            TimerStarted := TRUE;
        ELSIF DelayTimer.Q THEN
            IF CurrentState = STATE_OP THEN
                StateStep := 3; // Done
                TimerStarted := FALSE;
                DelayTimer(IN := FALSE);
            ELSE
                ErrorDetected := TRUE;
                ErrorMessage := 'Failed to enter OP';
            END_IF
        END_IF

    3: // Operational state reached
        // Normal operation or hold state
        EtherCAT_CMD := STATE_OP;

    ELSE
        // Invalid state
        ErrorDetected := TRUE;
        ErrorMessage := 'Invalid StateStep value';
END_CASE
