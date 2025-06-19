FUNCTION_BLOCK FB_ReadIOLinkValues
VAR_INPUT
    Trigger         : BOOL;        // Rising edge triggers the read sequence
    IndexList       : ARRAY[1..5] OF INT;  // IO-Link indexes for process data
    StartAddress    : INT;         // Base address or device ID of the IO-Link master
END_VAR

VAR_OUTPUT
    Value           : ARRAY[1..5] OF REAL;   // Retrieved process values
    ReadOK          : ARRAY[1..5] OF BOOL;   // TRUE if read was successful
    ErrorCode       : ARRAY[1..5] OF INT;    // 0 = OK, >0 = Error code
END_VAR

VAR
    i               : INT := 1;                  // Internal read counter
    Step            : INT := 0;                  // FSM step
    RisingEdge      : BOOL := FALSE;             // Rising edge detection
    PrevTrigger     : BOOL := FALSE;
    ReadBusy        : BOOL := FALSE;             // IO-Link read busy status
    TempRawValue    : REAL;                      // Temporary raw read value
    Timer           : TON;                       // Timeout timer
    TimeOut         : TIME := T#1s;              // Read timeout threshold
END_VAR


// Detect rising edge of Trigger
RisingEdge := Trigger AND NOT PrevTrigger;
PrevTrigger := Trigger;

CASE Step OF
    0: // Wait for Trigger
        IF RisingEdge THEN
            i := 1;
            Step := 1;
        END_IF

    1: // Start Read Request
        IF i <= 5 THEN
            // Simulate IOLinkReadRequest (replace with vendor-specific FB)
            ReadBusy := TRUE;
            Timer(IN := TRUE, PT := TimeOut);
            Step := 2;
        ELSE
            Step := 0; // All reads complete
        END_IF

    2: // Wait for Read Completion or Timeout
        Timer(IN := TRUE);

        // Simulated read response: Assume always successful for demo
        IF Timer.Q = FALSE THEN
            // Simulated response — replace with real read API
            TempRawValue := REAL_TO_REAL(i * 10.0); // Example: 10.0, 20.0, ...
            Value[i] := TempRawValue;
            ReadOK[i] := TRUE;
            ErrorCode[i] := 0;
            ReadBusy := FALSE;
            Timer(IN := FALSE);
            i := i + 1;
            Step := 1;
        ELSE
            // Timeout or Error
            ReadOK[i] := FALSE;
            ErrorCode[i] := 1001; // Custom timeout code
            Timer(IN := FALSE);
            i := i + 1;
            Step := 1;
        END_IF
END_CASE
