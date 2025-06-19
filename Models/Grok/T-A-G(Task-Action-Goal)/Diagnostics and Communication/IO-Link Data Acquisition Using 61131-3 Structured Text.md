(* Function Block: IO-Link Data Acquisition *)
FUNCTION_BLOCK FB_IOLinkDataAcquisition
VAR_INPUT
    Enable : BOOL;                     (* Enable function block *)
    IOLinkMasterIP : STRING[15];       (* IP address of IO-Link master *)
    PortNumber : UINT;                 (* Communication port *)
    TimeoutMs : UINT := 1000;          (* Communication timeout in ms *)
END_VAR

VAR_OUTPUT
    Values : ARRAY[0..4] OF REAL;      (* Acquired process values *)
    Status : ARRAY[0..4] OF BOOL;      (* Status of each value: TRUE = valid, FALSE = invalid *)
    Error : BOOL;                      (* Error flag *)
    ErrorID : UINT;                    (* Error code *)
    Active : BOOL;                     (* Function block is active *)
END_VAR

VAR
    CommState : UINT;                  (* Communication state: 0=Idle, 1=Connecting, 2=Connected, 3=Reading *)
    CommHandle : DWORD;                (* Communication handle *)
    RetryCount : UINT;                 (* Retry counter for communication *)
    MaxRetries : UINT := 3;            (* Maximum retry attempts *)
    DataBuffer : ARRAY[0..4] OF BYTE;  (* Raw data buffer *)
    LastError : UINT;                  (* Last communication error *)
    i : UINT;                          (* Loop counter *)
END_VAR

(* Initialize outputs *)
Active := FALSE;
Error := FALSE;
ErrorID := 0;

(* Main logic *)
IF Enable THEN
    Active := TRUE;
    
    CASE CommState OF
        0: (* Idle state *)
            IF IOLinkMasterIP <> '' AND PortNumber > 0 THEN
                (* Initiate connection *)
                CommHandle := SysCommOpen(IOLinkMasterIP, PortNumber, TimeoutMs);
                IF CommHandle <> 0 THEN
                    CommState := 1; (* Move to Connecting *)
                    RetryCount := 0;
                ELSE
                    Error := TRUE;
                    ErrorID := 100; (* Connection initialization failed *)
                    Active := FALSE;
                END_IF;
            ELSE
                Error := TRUE;
                ErrorID := 101; (* Invalid input parameters *)
                Active := FALSE;
            END_IF;

        1: (* Connecting *)
            IF SysCommIsConnected(CommHandle) THEN
                CommState := 2; (* Move to Connected *)
            ELSIF RetryCount < MaxRetries THEN
                RetryCount := RetryCount + 1;
                (* Retry connection *)
                CommHandle := SysCommOpen(IOLinkMasterIP, PortNumber, TimeoutMs);
            ELSE
                Error := TRUE;
                ErrorID := 102; (* Connection timeout *)
                CommState := 0;
                Active := FALSE;
            END_IF;

        2: (* Connected *)
            CommState := 3; (* Start reading data *)
            FOR i := 0 TO 4 DO
                Status[i] := FALSE; (* Initialize status *)
                Values[i] := 0.0;   (* Initialize values *)
            END_FOR;

        3: (* Reading data *)
            FOR i := 0 TO 4 DO
                (* Read each process value *)
                LastError := SysCommRead(CommHandle, i, DataBuffer[i], TimeoutMs);
                IF LastError = 0 THEN
                    (* Verify data integrity using CRC *)
                    IF CheckCRC(DataBuffer[i]) THEN
                        Values[i] := ConvertToReal(DataBuffer[i]);
                        Status[i] := TRUE;
                    ELSE
                        Status[i] := FALSE;
                        Error := TRUE;
                        ErrorID := 103 + i; (* CRC error for value i *)
                    END_IF;
                ELSE
                    Status[i] := FALSE;
                    Error := TRUE;
                    ErrorID := 200 + i; (* Read error for value i *)
                END_IF;
            END_FOR;
            
            (* All values processed, stay in connected state *)
            CommState := 2;

    END_CASE;

ELSE
    (* Disable function block *)
    IF Active THEN
        SysCommClose(CommHandle);
        CommState := 0;
        Active := FALSE;
        FOR i := 0 TO 4 DO
            Values[i] := 0.0;
            Status[i] := FALSE;
        END_FOR;
        Error := FALSE;
        ErrorID := 0;
    END_IF;
END_IF;

(* Helper function: CRC Check *)
METHOD CheckCRC : BOOL
VAR_INPUT
    Data : BYTE;
END_VAR
(* Simplified CRC check - replace with actual CRC algorithm *)
CheckCRC := (Data MOD 2 = 0); (* Example: Even parity check *)
END_METHOD

(* Helper function: Convert raw data to REAL *)
METHOD ConvertToReal : REAL
VAR_INPUT
    RawData : BYTE;
END_VAR
(* Example conversion - adjust based on actual data format *)
ConvertToReal := REAL_OF_BYTE(RawData) / 10.0;
END_METHOD

END_FUNCTION_BLOCK
