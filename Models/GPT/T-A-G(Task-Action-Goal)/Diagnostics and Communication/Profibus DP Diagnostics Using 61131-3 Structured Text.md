FUNCTION_BLOCK FB_ProfibusDP_Diagnostics
VAR_INPUT
    Execute       : BOOL;
    SlaveAddress  : BYTE;
    Timeout       : TIME;
END_VAR

VAR_OUTPUT
    Done          : BOOL;
    Busy          : BOOL;
    Error         : BOOL;
    ErrorID       : DWORD;
    DeviceStatus  : BYTE;
    CommState     : BOOL;
    ErrorCode     : BYTE;
END_VAR

VAR
    prevExecute   : BOOL := FALSE;
    state         : INT := 0;
    startTime     : TIME;
    elapsedTime   : TIME;
    response      : WORD;
    diagRaw       : WORD;
END_VAR

// Rising edge detection
IF Execute AND NOT prevExecute THEN
    Busy := TRUE;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    state := 1;
    startTime := TIME();
END_IF
prevExecute := Execute;

// Diagnostic state machine
CASE state OF

    1: // Send diagnostic request
        response := Profibus_ReadDiagnostic(SlaveAddress); // Stub call to read diagnostic word
        IF response <> 16#FFFF THEN
            diagRaw := response;
            state := 2;
        ELSE
            elapsedTime := TIME() - startTime;
            IF elapsedTime > Timeout THEN
                Error := TRUE;
                ErrorID := 1001; // Timeout error
                Busy := FALSE;
                state := 0;
            END_IF
        END_IF

    2: // Parse and report
        DeviceStatus := BYTE(diagRaw AND 16#00FF);
        ErrorCode := BYTE(SHR(diagRaw, 8));
        CommState := (DeviceStatus AND 16#01) <> 0;

        Done := TRUE;
        Busy := FALSE;
        Error := FALSE;
        ErrorID := 0;
        state := 0;

END_CASE

// Dummy Profibus read diagnostic stub
FUNCTION Profibus_ReadDiagnostic : WORD
VAR_INPUT
    addr : BYTE;
END_VAR
// Simulated valid diagnostic value
Profibus_ReadDiagnostic := 16#8701; // Status: 01 (Comm OK), Error: 87
