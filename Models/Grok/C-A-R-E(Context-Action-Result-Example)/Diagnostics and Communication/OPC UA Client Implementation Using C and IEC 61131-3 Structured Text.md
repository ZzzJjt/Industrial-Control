(* IEC 61131-3 Structured Text: OPC_UA_CLIENT Function Block *)
(* Purpose: Manages OPC UA client connection to a server using open62541 C library *)

FUNCTION_BLOCK OPC_UA_CLIENT
VAR_INPUT
    Execute : BOOL;                 (* Trigger connection on rising edge *)
    ServerUrl : STRING[255];        (* OPC UA server URL, e.g., opc.tcp://192.168.1.100:4840 *)
    Timeout : TIME;                 (* Connection timeout, e.g., T#5s *)
END_VAR
VAR_OUTPUT
    Done : BOOL;                    (* TRUE on successful connection *)
    Busy : BOOL;                    (* TRUE during connection attempt *)
    Error : BOOL;                   (* TRUE if error occurs *)
    ErrorID : DWORD;                (* 0: No error, 0x01: Invalid URL, 0x02: Timeout, 0x03: Server Unavailable, 0x04: Connection Refused, 0x05: Internal Error *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Idle, 1=Connect, 2=Wait, 3=Complete *)
    LastExecute : BOOL;             (* Previous Execute state for edge detection *)
    Timer : TON;                    (* Timeout timer *)
    ClientPtr : DWORD;              (* Pointer to UA_Client, managed by C *)
    C_Result : INT;                 (* C function return code *)
END_VAR

(* External C function declarations *)
EXTERNAL
    FUNCTION opc_ua_connect : INT
        VAR_INPUT
            server_url : STRING[255];
            timeout_ms : INT;
            client : DWORD;
        END_VAR
    END_FUNCTION

    FUNCTION opc_ua_disconnect
        VAR_INPUT
            client : DWORD;
        END_VAR
    END_FUNCTION
END_EXTERNAL

(* Main Logic *)
IF NOT Execute AND NOT LastExecute THEN
    (* Reset outputs when idle *)
    Done := FALSE;
    Busy := FALSE;
    Error := FALSE;
    ErrorID := 0;
    State := 0;
    Timer.IN := FALSE;
    IF ClientPtr <> 0 THEN
        opc_ua_disconnect(ClientPtr);
        ClientPtr := 0;
    END_IF;
ELSE
    CASE State OF
        0: (* Idle *)
            IF Execute AND NOT LastExecute THEN (* Rising edge *)
                (* Validate ServerUrl *)
                IF LEN(ServerUrl) = 0 OR LEN(ServerUrl) > 255 THEN
                    Error := TRUE;
                    ErrorID := 16#01; (* Invalid URL *)
                    Done := TRUE;
                    State := 0;
                ELSE
                    Busy := TRUE;
                    State := 1; (* Connect *)
                END_IF;
            END_IF;
        
        1: (* Connect *)
            (* Call C function *)
            C_Result := opc_ua_connect(ServerUrl, TIME_TO_INT(Timeout), ClientPtr);
            Timer(IN := TRUE, PT := Timeout);
            State := 2; (* Wait *)
        
        2: (* Wait *)
            IF C_Result = 0 THEN
                Done := TRUE;
                Busy := FALSE;
                State := 3; (* Complete *)
                Timer.IN := FALSE;
            ELSIF C_Result IN {1, 2, 3, 4, 5} THEN
                Error := TRUE;
                ErrorID := C_Result; (* Map C result to ErrorID *)
                Done := TRUE;
                Busy := FALSE;
                State := 0;
                Timer.IN := FALSE;
                IF ClientPtr <> 0 THEN
                    opc_ua_disconnect(ClientPtr);
                    ClientPtr := 0;
                END_IF;
            ELSIF Timer.Q THEN
                Error := TRUE;
                ErrorID := 16#02; (* Timeout *)
                Done := TRUE;
                Busy := FALSE;
                State := 0;
                IF ClientPtr <> 0 THEN
                    opc_ua_disconnect(ClientPtr);
                    ClientPtr := 0;
                END_IF;
            END_IF;
        
        3: (* Complete *)
            (* Remain connected until Execute falls or reset *)
            IF NOT Execute THEN
                Done := FALSE;
                Busy := FALSE;
                State := 0;
                IF ClientPtr <> 0 THEN
                    opc_ua_disconnect(ClientPtr);
                    ClientPtr := 0;
                END_IF;
            END_IF;
    END_CASE;
END_IF;

LastExecute := Execute;

(* Notes:
   - Purpose: OPC UA client for connecting to a server using open62541 C library.
   - Inputs:
     - Execute: Triggers connection on rising edge.
     - ServerUrl: OPC UA server URL (STRING[255], e.g., opc.tcp://192.168.1.100:4840).
     - Timeout: Connection timeout (e.g., T#5s).
   - Outputs:
     - Done: TRUE on successful connection.
     - Busy: TRUE during connection attempt.
     - Error: TRUE if error occurs.
     - ErrorID: 0 (No error), 0x01 (Invalid URL), 0x02 (Timeout), 0x03 (Server Unavailable), 0x04 (Connection Refused), 0x05 (Internal Error).
   - Logic:
     - State 0 (Idle): Resets outputs, awaits Execute.
     - State 1 (Connect): Validates ServerUrl, calls C function opc_ua_connect.
     - State 2 (Wait): Processes C result, applies timeout.
     - State 3 (Complete): Maintains connection until Execute falls.
   - Error Handling:
     - Invalid URL: Empty or >255 chars (ErrorID=0x01).
     - Timeout: Connection exceeds Timeout (ErrorID=0x02).
     - Server Unavailable: Server not found or closed (ErrorID=0x03).
     - Connection Refused: Server rejects connection (ErrorID=0x04).
     - Internal Error: Library or memory issues (ErrorID=0x05).
   - Safety:
     - Rising edge detection prevents repeated triggers.
     - Bounded operations (STRING[255], timer) ensure scan-cycle safety (e.g., 10–100 ms cycles).
     - Cleans up client on error or disable.
   - Integration:
     - Calls C functions opc_ua_connect, opc_ua_disconnect from open62541 library.
     - ClientPtr managed by C, persists during connection.
   - Usage:
     - Production line: Execute=TRUE, ServerUrl="opc.tcp://192.168.1.100:4840", Timeout=T#5s.
     - Success: Done=TRUE, enables data exchange.
     - Timeout: Error=TRUE, ErrorID=0x02, triggers retry or alarm.
   - Maintenance:
     - Modular design aids debugging.
     - ErrorID supports HMI diagnostics.
   - Platform Notes:
     - Assumes STRING[255], TIME, and C integration (e.g., CODESYS, TwinCAT).
     - Timeout in milliseconds for C; adjust for platform-specific TIME handling.
     - open62541 library must be linked; replace opc_ua_connect with platform-specific wrapper if needed.
*)
END_FUNCTION_BLOCK
