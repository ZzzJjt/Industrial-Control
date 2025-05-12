(* Function Block: PROFIBUS_DP_DIAGNOSTICS
   Purpose: Retrieves diagnostic information from a Profibus DP slave device.
   Features:
   - Initiates diagnostic request on rising edge of EXECUTE or cyclically if CYCLE is TRUE
   - Retrieves device status, error codes, and communication health
   - Provides structured outputs for monitoring
   - Includes error handling for timeouts and invalid responses
*)

FUNCTION_BLOCK PROFIBUS_DP_DIAGNOSTICS
VAR_INPUT
    EXECUTE : BOOL;               (* Rising edge triggers one-shot diagnostic request *)
    CYCLE : BOOL;                 (* TRUE enables cyclic operation *)
    SLAVE_ADDRESS : BYTE;         (* Profibus slave address, 0-125 *)
    TIMEOUT : TIME := T#1s;       (* Timeout for diagnostic request *)
    CYCLE_INTERVAL : TIME := T#2s; (* Interval for cyclic requests *)
END_VAR

VAR_OUTPUT
    DONE : BOOL;                  (* TRUE when request completes successfully *)
    BUSY : BOOL;                  (* TRUE during request processing *)
    ERROR : BOOL;                 (* TRUE if request fails *)
    ERROR_ID : UINT;              (* 0=None, 1=Invalid Address, 2=Timeout, 3=Comm Error, 4=Invalid Response *)
    DEVICE_STATUS : BYTE;         (* Device status: 0=OK, 1=Fault, 2=Config Error, 3=Not Ready *)
    COMM_STATUS : BYTE;           (* Comm status: 0=OK, 1=Link Down, 2=Data Error *)
    ERROR_CODE : UINT;            (* Slave-specific error code *)
    EXT_DIAG_DATA : ARRAY[0..63] OF BYTE; (* Extended diagnostic data buffer *)
END_VAR

VAR
    LastExecute : BOOL;           (* Tracks previous EXECUTE state for edge detection *)
    CycleTimer : TON;             (* Timer for cyclic operation *)
    RequestTimer : TON;           (* Timer for request timeout *)
    DiagnosticBuffer : ARRAY[0..255] OF BYTE; (* Raw diagnostic response buffer *)
    BufferLength : UINT;          (* Length of received diagnostic data *)
    RequestActive : BOOL;         (* Flag for active request *)
    C_Result : UINT;              (* Result from Profibus interface *)
END_VAR

(* Initialize outputs *)
DONE := FALSE;
BUSY := FALSE;
ERROR := FALSE;
ERROR_ID := 0;
DEVICE_STATUS := 0;
COMM_STATUS := 0;
ERROR_CODE := 0;
FOR i := 0 TO 63 DO
    EXT_DIAG_DATA[i] := 0;
END_FOR

(* Main logic *)
(* Start request on EXECUTE rising edge or cyclic trigger *)
IF (EXECUTE AND NOT LastExecute) OR (CYCLE AND CycleTimer.Q AND NOT RequestActive) THEN
    (* Initialize request *)
    BUSY := TRUE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    RequestActive := TRUE;
    DEVICE_STATUS := 0;
    COMM_STATUS := 0;
    ERROR_CODE := 0;
    FOR i := 0 TO 63 DO
        EXT_DIAG_DATA[i] := 0;
    END_FOR
    
    (* Validate slave address *)
    IF SLAVE_ADDRESS > 125 THEN
        ERROR := TRUE;
        ERROR_ID := 1; (* Invalid Address *)
        BUSY := FALSE;
        RequestActive := FALSE;
        RETURN;
    END_IF
    
    (* Start diagnostic request *)
    RequestTimer(IN := TRUE, PT := TIMEOUT);
    C_Result := PB_DP_GetDiagnostics(SLAVE_ADDRESS, DiagnosticBuffer, ADR(BufferLength));
END_IF

(* Process response *)
IF RequestActive AND (C_Result <> 255 OR RequestTimer.Q) THEN
    IF RequestTimer.Q THEN
        (* Timeout *)
        ERROR := TRUE;
        ERROR_ID := 2; (* Timeout *)
        BUSY := FALSE;
        RequestActive := FALSE;
        RequestTimer(IN := FALSE);
    ELSIF C_Result <> 0 THEN
        (* Communication error *)
        ERROR := TRUE;
        ERROR_ID := 3; (* Comm Error *)
        BUSY := FALSE;
        RequestActive := FALSE;
        RequestTimer(IN := FALSE);
    ELSE
        (* Parse diagnostic buffer *)
        IF BufferLength >= 6 THEN
            (* Standard diagnostic data (first 6 bytes) *)
            DEVICE_STATUS := DiagnosticBuffer[0] AND 16#0F; (* Lower 4 bits: status *)
            COMM_STATUS := (DiagnosticBuffer[1] >> 4) AND 16#0F; (* Upper 4 bits: comm status *)
            ERROR_CODE := (DiagnosticBuffer[4] << 8) OR DiagnosticBuffer[5]; (* Error code *)
            
            (* Extended diagnostic data *)
            IF BufferLength > 6 THEN
                FOR i := 0 TO MIN(BufferLength - 7, 63) DO
                    EXT_DIAG_DATA[i] := DiagnosticBuffer[6 + i];
                END_FOR
            END_IF
            
            DONE := TRUE;
            BUSY := FALSE;
            RequestActive := FALSE;
            RequestTimer(IN := FALSE);
        ELSE
            (* Invalid response *)
            ERROR := TRUE;
            ERROR_ID := 4; (* Invalid Response *)
            BUSY := FALSE;
            RequestActive := FALSE;
            RequestTimer(IN := FALSE);
        END_IF
    END_IF
END_IF

(* Manage cyclic operation *)
IF CYCLE THEN
    CycleTimer(IN := TRUE, PT := CYCLE_INTERVAL);
ELSE
    CycleTimer(IN := FALSE);
END_IF

(* Reset outputs when neither EXECUTE nor CYCLE is active *)
IF NOT EXECUTE AND NOT CYCLE THEN
    BUSY := FALSE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_ID := 0;
    RequestActive := FALSE;
    RequestTimer(IN := FALSE);
    CycleTimer(IN := FALSE);
END_IF

(* Update edge detection *)
LastExecute := EXECUTE;

(* Simulated Profibus diagnostic function *)
FUNCTION PB_DP_GetDiagnostics : UINT
VAR_INPUT
    SlaveAddress : BYTE;
    Buffer : ARRAY[0..255] OF BYTE;
    BufferLength : DWORD; (* Pointer to length *)
END_VAR
(* Placeholder: Simulates Profibus DP diagnostic request *)
(* Returns: 0=Success, 255=Pending, other=Error *)
(* Simulate valid response *)
Buffer[0] := 0; (* Device OK *)
Buffer[1] := 0; (* Comm OK *)
Buffer[4] := 0; (* Error code high *)
Buffer[5] := 0; (* Error code low *)
BufferLength := 6; (* Standard diagnostic length *)
PB_DP_GetDiagnostics := 0;
END_FUNCTION
END_FUNCTION_BLOCK
