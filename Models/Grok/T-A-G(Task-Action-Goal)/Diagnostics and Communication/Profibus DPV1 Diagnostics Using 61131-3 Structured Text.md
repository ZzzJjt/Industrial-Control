(* Program: Profibus DPV1 Diagnostic Retrieval and Interpretation *)
PROGRAM PRG_ProfibusDPV1Diag
VAR
    Execute : BOOL;                    (* Trigger diagnostic request *)
    SlaveAddress : BYTE;               (* Profibus DPV1 slave address (1-126) *)
    Timeout : TIME := T#1000ms;        (* Timeout for communication *)
    Done : BOOL;                       (* Diagnostic request completed *)
    Busy : BOOL;                       (* Operation in progress *)
    Error : BOOL;                      (* Error occurred *)
    ErrorID : DWORD;                   (* Detailed error code *)
    DiagData : DiagStructure;          (* Structured diagnostic data *)
    State : UINT;                      (* State: 0=Idle, 1=Requesting, 2=Parsing *)
    LastExecute : BOOL;                (* For rising edge detection *)
    TimeoutTimer : TON;                (* Timeout timer *)
    DiagBuffer : ARRAY[0..255] OF BYTE; (* Raw diagnostic buffer *)
    DiagLen : UINT;                    (* Length of received diagnostic data *)
    CommHandle : DWORD;                (* Communication handle *)
END_VAR

(* Diagnostic data structure *)
TYPE DiagStructure :
STRUCT
    CommError : BOOL;                  (* Communication error detected *)
    DeviceStatus : BYTE;               (* Device status: 0=OK, 1=Fault, etc. *)
    ParamFault : BOOL;                 (* Parameter configuration fault *)
    ConfigIssue : BOOL;                (* Configuration mismatch *)
    PowerSupply : BOOL;                (* Power supply issue *)
    HardwareFailure : BOOL;            (* Hardware failure detected *)
    BusInterruption : BOOL;            (* Bus communication interrupted *)
    WatchdogTimeout : BOOL;            (* Watchdog timeout occurred *)
    TempAlert : BOOL;                  (* Temperature out of range *)
    ManufSpecific : ARRAY[0..31] OF BYTE; (* Manufacturer-specific diagnostics *)
    ManufSpecLen : UINT;               (* Length of manufacturer-specific data *)
END_STRUCT
END_TYPE

(* Initialize outputs *)
Done := FALSE;
Busy := FALSE;
Error := FALSE;
ErrorID := 0;
DiagData.CommError := FALSE;
DiagData.DeviceStatus := 0;
DiagData.ParamFault := FALSE;
DiagData.ConfigIssue := FALSE;
DiagData.PowerSupply := FALSE;
DiagData.HardwareFailure := FALSE;
DiagData.BusInterruption := FALSE;
DiagData.WatchdogTimeout := FALSE;
DiagData.TempAlert := FALSE;
DiagData.ManufSpecLen := 0;

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
            FOR i : UINT := 0 TO 31 DO
                DiagData.ManufSpecific[i] := 0;
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
        (* Initiate DPV1 diagnostic request *)
        CommHandle := SysProfibusDPV1_DiagRequest(SlaveAddress, DiagBuffer, 256, Timeout);
        IF CommHandle <> 0 THEN
            DiagLen := SysProfibusDPV1_GetDiagData(CommHandle, DiagBuffer);
            IF DiagLen >= 6 THEN (* Minimum DPV1 diagnostic length *)
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
            DiagData.CommError := TRUE;
        END_IF;

        (* Check for timeout *)
        IF TimeoutTimer.Q THEN
            Error := TRUE;
            ErrorID := 16#80040000; (* Timeout *)
            State := 0;
            Busy := FALSE;
            DiagData.CommError := TRUE;
        END_IF;

    2: (* Parsing *)
        (* Parse diagnostic data using CASE for 10 diagnostic types *)
        FOR i : UINT := 0 TO DiagLen - 1 DO
            CASE DiagBuffer[i] OF
                16#01: (* Communication Error *)
                    DiagData.CommError := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#02: (* Device Status *)
                    IF i + 1 < DiagLen THEN
                        DiagData.DeviceStatus := DiagBuffer[i + 1];
                        i := i + 1;
                    ELSE
                        Error := TRUE;
                        ErrorID := 16#80050000; (* Invalid data *)
                    END_IF;

                16#03: (* Parameter Fault *)
                    DiagData.ParamFault := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#04: (* Configuration Issue *)
                    DiagData.ConfigIssue := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#05: (* Power Supply Problem *)
                    DiagData.PowerSupply := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#06: (* Hardware Failure *)
                    DiagData.HardwareFailure := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#07: (* Bus Interruption *)
                    DiagData.BusInterruption := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#08: (* Watchdog Timeout *)
                    DiagData.WatchdogTimeout := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#09: (* Temperature Alert *)
                    DiagData.TempAlert := TRUE;
                    IF i + 1 < DiagLen THEN
                        i := i + 1; (* Skip detail byte *)
                    END_IF;

                16#0A: (* Manufacturer-Specific Diagnostics *)
                    IF i + 1 < DiagLen THEN
                        DiagData.ManufSpecLen := MIN(DiagLen - i - 1, 32);
                        FOR j : UINT := 0 TO DiagData.ManufSpecLen - 1 DO
                            IF i + 1 + j < DiagLen THEN
                                DiagData.ManufSpecific[j] := DiagBuffer[i + 1 + j];
                            END_IF;
                        END_FOR;
                        i := i + DiagData.ManufSpecLen;
                    ELSE
                        Error := TRUE;
                        ErrorID := 16#80060000; (* Invalid manufacturer data *)
                    END_IF;

                ELSE
                    (* Unsupported diagnostic type *)
                    Error := TRUE;
                    ErrorID := 16#80070000; (* Unsupported diagnostic type *)
                    i := DiagLen; (* Exit loop *)
            END_CASE;
        END_FOR;

        (* Validate and finalize *)
        IF NOT Error THEN
            Done := TRUE;
            Busy := FALSE;
            State := 0;
        ELSE
            State := 0;
            Busy := FALSE;
        END_IF;

END_CASE;

(* Cleanup on falling edge or error *)
IF NOT Execute OR Error THEN
    IF State <> 0 THEN
        SysProfibusDPV1_CloseHandle(CommHandle);
        CommHandle := 0;
        State := 0;
        Busy := FALSE;
        IF NOT Error THEN
            Done := FALSE;
        END_IF;
    END_IF;
END_IF;

(* External function declarations - replace with actual Profibus DPV1 library calls *)
EXTERNAL
    FUNCTION SysProfibusDPV1_DiagRequest : DWORD
        VAR_INPUT
            address : BYTE;
            buffer : ARRAY[0..255] OF BYTE;
            maxLen : UINT;
            timeout : TIME;
        END_VAR
    END_FUNCTION
    FUNCTION SysProfibusDPV1_GetDiagData : UINT
        VAR_INPUT
            handle : DWORD;
            buffer : ARRAY[0..255] OF BYTE;
        END_VAR
    END_FUNCTION
    FUNCTION SysProfibusDPV1_CloseHandle
        VAR_INPUT
            handle : DWORD;
        END_VAR
    END_FUNCTION
END_EXTERNAL

END_PROGRAM
