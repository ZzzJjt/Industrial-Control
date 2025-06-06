FUNCTION_BLOCK FB_EtherCATStateManager
VAR_INPUT
    CurrentState       : INT;       // Actual state read from EtherCAT slave
    RequestedStart     : BOOL;      // Trigger to start ESM sequence
END_VAR

VAR_OUTPUT
    DesiredState       : INT;       // State to be written to control register
    Error              : BOOL;      // Set TRUE on invalid transition
    ErrorMsg           : STRING;    // Descriptive error message
    InTransition       : BOOL;      // Indicates transition is in progress
END_VAR

VAR
    StateStep          : INT := 0;      // Internal sequence tracker
    TransitionTimer    : TON;           // 5-second delay timer
    TimerStarted       : BOOL := FALSE;
END_VAR

// --- Symbolic EtherCAT State Definitions ---
VAR CONSTANT
    STATE_INIT   : INT := 1;
    STATE_PREOP  : INT := 2;
    STATE_SAFEOP : INT := 4;
    STATE_OP     : INT := 8;
END_VAR

// --- Timer instance configuration ---
TransitionTimer(IN := TimerStarted, PT := T#5s);

// --- Main Control Logic ---
IF RequestedStart THEN
    InTransition := TRUE;

    CASE StateStep OF
        0: // Transition to PREOP
            IF CurrentState = STATE_INIT THEN
                DesiredState := STATE_PREOP;
                TimerStarted := TRUE;
                IF TransitionTimer.Q THEN
                    StateStep := 1;
                    TimerStarted := FALSE;
                    TransitionTimer(IN := FALSE); // reset timer
                END_IF
            ELSE
                Error := TRUE;
                ErrorMsg := 'Expected INIT state.';
            END_IF

        1: // Transition to SAFEOP
            IF CurrentState = STATE_PREOP THEN
                DesiredState := STATE_SAFEOP;
                TimerStarted := TRUE;
                IF TransitionTimer.Q THEN
                    StateStep := 2;
                    TimerStarted := FALSE;
                    TransitionTimer(IN := FALSE);
                END_IF
            ELSE
                Error := TRUE;
                ErrorMsg := 'Expected PREOP state.';
            END_IF

        2: // Transition to OP
            IF CurrentState = STATE_SAFEOP THEN
                DesiredState := STATE_OP;
                TimerStarted := TRUE;
                IF TransitionTimer.Q THEN
                    StateStep := 3;
                    TimerStarted := FALSE;
                    TransitionTimer(IN := FALSE);
                END_IF
            ELSE
                Error := TRUE;
                ErrorMsg := 'Expected SAFEOP state.';
            END_IF

        3: // Final OP state reached
            IF CurrentState = STATE_OP THEN
                InTransition := FALSE;
            ELSE
                Error := TRUE;
                ErrorMsg := 'Transition to OP failed.';
            END_IF
    END_CASE

ELSE
    StateStep := 0;
    DesiredState := STATE_INIT;
    TimerStarted := FALSE;
    InTransition := FALSE;
    TransitionTimer(IN := FALSE);
    Error := FALSE;
    ErrorMsg := '';
END_IF

EtherCATManager(
    CurrentState := ReadSlaveState(),
    RequestedStart := StartSequence
);

WriteControlWord(EtherCATManager.DesiredState);
IF EtherCATManager.Error THEN
    DisplayAlarm(EtherCATManager.ErrorMsg);
END_IF;
