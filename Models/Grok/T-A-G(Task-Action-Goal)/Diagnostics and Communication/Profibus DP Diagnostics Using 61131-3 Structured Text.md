(* Function Block: Profibus DP Slave Diagnostic *)
FUNCTION_BLOCK FB_ProfibusDPDiag
VAR_INPUT
    Execute : BOOL;                    (* Trigger diagnostic request on rising edge *)
    SlaveAddress : BYTE;               (* Profibus DP slave address (1-126) *)
    Timeout : TIME;                    (* Timeout for communication *)
END_VAR

VAR_OUTPUT
    Done : BOOL;                       (* Diagnostic request completed successfully *)
    Busy : BOOL;                       (* Operation in progress *)
    Error : BOOL;                      (* Error occurred *)
    ErrorID : DWORD;                   (* Detailed error code *)
    DeviceStatus : BYTE;               (* Device status: 0=OK, 1=ConfigFault, 2=ExtDiag, etc. *)
    CommState : BYTE;                  (* Communication state: 0=OK, 1=Timeout, 2=Lost *)
    ExtDiagData : ARRAY[0..63] OF BYTE; (* Extended diagnostic data buffer *)
    ExtDiagLen : UINT;                 (* Length of valid extended diagnostic data *)
END_VAR

VAR
    State : UINT;                      (* State: 0=Idle, 1=Requesting, 2=Parsing *)
    LastExecute : BOOL;                (* For rising edge detection *)
    TimeoutTimer : TON;                (* Timeout timer *)
    DiagBuffer : ARRAY[0..255] OF BYTE; (* Raw diagnostic buffer *)
    DiagLen : UINT;                    (* Length of received diagnostic data *)
    CommHandle : DWORD;                (* Communication handle *)
END_VAR

(* Initialize outputs *)
Done := FALSE;
Busy := FALSE;
Error := FALSE;
ErrorID := 0;
DeviceStatus := 0;
CommState := 0;
ExtDiagLen := 0;

(* Timeout timer logic *)
TimeoutTimer(IN := Busy AND State <> 0, PT := Timeout);

(* Main logic *)
IF Execute AND NOT LastExecute THEN
    (* Rising edge detected *)
    IF State = 0 THEN
        IF SlaveAddress >= 1 AND SlaveAddress <= 126 THEN
            Busy := TRUE;
            State := 1; (* Move to Requesting *)
            Error := FALSE;
            ErrorID := 0;
            Done := FALSE;
            DiagLen := 0;
            ExtDiagLen := 0;
            FOR i : UINT := 0 TO 63 DO
                ExtDiagData[i] := 0;
            END_FOR;
        ELSE
            Error := TRUE;
            ErrorID := 16#80010000; (* Invalid slave address *)
        END_IF;
    END_IF;
END_IF;

LastExecute := Execute;

CASE State OF
    0: (* Idle *)
        (* Nothing to do *)

    1: (* Requesting *)
        (* Initiate diagnostic request *)
        CommHandle := SysProfibusDP_DiagRequest(SlaveAddress, DiagBuffer, 256, Timeout);
        IF CommHandle <> 0 THEN
            DiagLen := SysProfibusDP_GetDiagData(CommHandle, DiagBuffer);
            IF DiagLen >= 6 THEN (* Minimum standard diagnostic length *)
                State := 2; (* Move to Parsing *)
            ELSE
                Error := TRUE;
                ErrorID := 16#80020000; (* Incomplete diagnostic data *)
                State := 0;
                Busy := FALSE;
            END_IF;
        ELSE
            Error := TRUE;
            ErrorID := 16#80030000; (* Communication failure *)
            State := 0;
            Busy := FALSE;
        END_IF;

        (* Check for timeout *)
        IF TimeoutTimer.Q THEN
            Error := TRUE;
            ErrorID := 16#80040000; (* Timeout *)
            State := 0;
            Busy := FALSE;
            CommState := 1; (* Timeout *)
        END_IF;

    2: (* Parsing *)
        (* Parse standard diagnostic data *)
        DeviceStatus := DiagBuffer[0]; (* Status byte 1 *)
        CommState := DiagBuffer[1] AND 16#0F; (* Lower nibble of status byte 2 *)
        
        (* Check for extended diagnostics *)
        IF DiagLen > 6 THEN
            ExtDiagLen := MIN(DiagLen - 6, 64); (* Limit to output array size *)
            FOR i : UINT := 0 TO ExtDiagLen - 1 DO
                ExtDiagData[i] := DiagBuffer[6 + i];
            END_FOR;
        END_IF;

        (* Validate diagnostic data *)
        IF DeviceStatus <= 4 THEN (* Known status codes: 0=OK, 1=ConfigFault, etc. *)
            Done := TRUE;
            Busy := FALSE;
            State := 0;
        ELSE
            Error := TRUE;
            ErrorID := 16#80050000; (* Invalid diagnostic data *)
            State := 0;
            Busy := FALSE;
        END_IF;

END_CASE;

(* Cleanup on falling edge or error *)
IF NOT Execute OR Error THEN
    IF State <> 0 THEN
        SysProfibusDP_CloseHandle(CommHandle);
        CommHandle := 0;
        State := 0;
        Busy := FALSE;
        IF NOT Error THEN
            Done := FALSE;
        END_IF;
    END_IF;
END_IF;

(* External function declarations - replace with actual Profibus DP library calls *)
EXTERNAL
    FUNCTION SysProfibusDP_DiagRequest : DWORD
        VAR_INPUT
            address : BYTE;
            buffer : ARRAY[0..255] OF BYTE;
            maxLen : UINT;
            timeout : TIME;
        END_VAR
    END_FUNCTION
    FUNCTION SysProfibusDP_GetDiagData : UINT
        VAR_INPUT
            handle : DWORD;
            buffer : ARRAY[0..255] OF BYTE;
        END_VAR
    END_FUNCTION
    FUNCTION SysProfibusDP_CloseHandle
        VAR_INPUT
            handle : DWORD;
        END_VAR
    END_FUNCTION
END_EXTERNAL

END_FUNCTION_BLOCK
