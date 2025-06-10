FUNCTION_BLOCK FB_ProfibusDiagRead
VAR_INPUT
    Execute       : BOOL;         // Rising-edge trigger to start diagnostic read
    SlaveAddress  : BYTE;         // Profibus slave address
    Timeout       : TIME;         // Timeout duration
END_VAR

VAR_OUTPUT
    Done          : BOOL;         // TRUE if diagnostics are successfully retrieved
    Busy          : BOOL;         // TRUE while request is in progress
    Error         : BOOL;         // TRUE if error occurred
    ErrorID       : DWORD;        // Diagnostic error code
    DeviceStatus  : BYTE;         // Parsed device status
    CommHealth    : BYTE;         // Parsed communication health
END_VAR

VAR
    rTrig         : R_TRIG;
    StartTime     : TIME;
    Timer         : TON;
    State         : INT := 0;
    DiagBuffer    : ARRAY[0..15] OF BYTE; // Simulated response buffer
    CommResult    : BOOL; // Simulated response flag
END_VAR

// Simulated function (to be replaced with actual bus call)
FUNCTION SimulatedProfibusDiagRead : BOOL
VAR_INPUT
    Addr : BYTE;
    pBuffer : POINTER TO BYTE;
END_VAR
// Returns TRUE if read succeeded and fills pBuffer^
BEGIN
    pBuffer^[0] := 16#AA; // Device status example
    pBuffer^[1] := 16#55; // Comm health example
    RETURN TRUE; // Simulate success
END_FUNCTION

// Detect rising edge
rTrig(CLK := Execute);

CASE State OF

0: // Idle
    IF rTrig.Q THEN
        Busy := TRUE;
        Done := FALSE;
        Error := FALSE;
        ErrorID := 0;
        Timer(IN := TRUE, PT := Timeout);
        StartTime := TIME(); // Record start
        State := 10;
    END_IF

10: // Send diagnostic request
    CommResult := SimulatedProfibusDiagRead(Addr := SlaveAddress, pBuffer := ADR(DiagBuffer[0]));
    IF CommResult THEN
        State := 20;
    ELSIF Timer.Q THEN
        State := 100; // Timeout
    END_IF

20: // Parse response
    DeviceStatus := DiagBuffer[0];
    CommHealth   := DiagBuffer[1];
    Done := TRUE;
    Busy := FALSE;
    State := 0;

100: // Timeout/Error
    Error := TRUE;
    Busy := FALSE;
    ErrorID := 16#00010001; // Example error code
    State := 0;
END_CASE
