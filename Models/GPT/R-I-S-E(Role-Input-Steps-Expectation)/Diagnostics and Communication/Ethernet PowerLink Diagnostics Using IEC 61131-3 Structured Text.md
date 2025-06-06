FUNCTION_BLOCK FB_PowerLinkDiagnostics
VAR_INPUT
    NodeID         : INT;     // Identifier of the PowerLink node to query
    Trigger        : BOOL;    // Rising edge triggers the diagnostic request
    CommStatusIn   : BOOL;    // Input from PowerLink stack indicating MN availability
    RawDiagData    : DWORD;   // Raw diagnostic register data from MN (simulated/input)
END_VAR

VAR_OUTPUT
    NodeReachable  : BOOL;    // TRUE if node is online/responding
    ErrorCode      : WORD;    // Diagnostic error code reported by node
    NodeHealth     : INT;     // Health level (0 = OK, 1 = Warning, 2 = Fault)
    CommStatusOut  : BOOL;    // TRUE if MN communication is confirmed
    AlertFlag      : BOOL;    // TRUE if critical error is detected
    LastPollTime   : TIME;    // Time of last successful poll
END_VAR

VAR
    RisingEdge     : R_TRIG;
    TimerPoll      : TON;
    PollCycleTime  : TIME := T#1s;     // Poll every 1 second
    PollRequest    : BOOL := FALSE;
    SystemTime     : TIME;
END_VAR

// --- Rising edge detection to initiate polling ---
RisingEdge(CLK := Trigger);

// --- Cyclic polling timer ---
TimerPoll(IN := TRUE, PT := PollCycleTime);
IF TimerPoll.Q THEN
    PollRequest := TRUE;
    TimerPoll(IN := FALSE);     // Reset timer
END_IF

// --- Polling logic and diagnostic parsing ---
IF PollRequest OR RisingEdge.Q THEN
    CommStatusOut := CommStatusIn;

    IF CommStatusIn THEN
        // Simulate MN diagnostic read (RawDiagData already provided)
        NodeReachable := (SHR(RawDiagData, 31) = 0);   // Assume MSB indicates online status
        ErrorCode := WORD_TO_INT(SHR(RawDiagData, 16) AND 16#FFFF); // Error code bits
        NodeHealth := INT_TO_BYTE(RawDiagData AND 16#FF);           // Health indicator bits

        // Set AlertFlag if health indicates fault or communication loss
        AlertFlag := (NodeHealth = 2) OR (NOT NodeReachable);

        // Update timestamp
        LastPollTime := TIME(); // Replace with system time source if available
    ELSE
        // MN unreachable
        NodeReachable := FALSE;
        ErrorCode := 9999;
        NodeHealth := 2;
        AlertFlag := TRUE;
    END_IF

    PollRequest := FALSE;
    TimerPoll(IN := TRUE); // Restart timer
END_IF
