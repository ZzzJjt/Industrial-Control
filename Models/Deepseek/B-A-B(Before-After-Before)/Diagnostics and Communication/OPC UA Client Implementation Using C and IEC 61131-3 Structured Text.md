/* opcua_client.h */
#include <open62541/client.h>
#include <open62541/client_config_default.h>

typedef struct {
    UA_Boolean isConnected;
    UA_StatusCode lastError;
    UA_Client *client;
} OPCUA_Connection;

UA_StatusCode OPCUA_Connect(OPCUA_Connection *conn, 
                           const char *serverUrl, 
                           UA_UInt32 timeoutMs);
void OPCUA_Disconnect(OPCUA_Connection *conn);

/* plc_interface.c */
#pragma once
#ifdef CODESYS
    #include <bur/plctypes.h>
#elif defined(TWINCAT)
    #include <TcNative.h>
#endif

// Wrapper for PLC memory management
DLLEXPORT int OPCUA_EstablishConnection(
    char* serverUrl,
    unsigned long timeoutMs,
    unsigned long* errorCode)
{
    OPCUA_Connection conn = {0};
    UA_StatusCode ret = OPCUA_Connect(&conn, serverUrl, timeoutMs);
    
    *errorCode = (unsigned long)ret;
    return (ret == UA_STATUSCODE_GOOD) ? 1 : 0;
}

FUNCTION_BLOCK OPCUA_Connector
VAR_INPUT
    Execute : BOOL;
    ServerUrl : STRING(255);
    Timeout : TIME := T#5S;
END_VAR

VAR_OUTPUT
    Done : BOOL;
    Busy : BOOL;
    Error : BOOL;
    ErrorID : DWORD;
END_VAR

VAR
    {attribute 'hidden'}
    internalState : INT;
    {attribute 'hidden'}
    prevExecute : BOOL;
    {attribute 'externallink':='opcua_client.dll'}
    {attribute 'functionprefix':=''}
    Connect : DWORD := 0; (* External C function reference *)
END_VAR

METHOD EstablishConnection : BOOL
VAR_TEMP
    executeEdge : BOOL;
    extResult : DWORD;
END_VAR

executeEdge := Execute AND NOT prevExecute;
prevExecute := Execute;

CASE internalState OF
    0: (* Idle *)
        IF executeEdge THEN
            Busy := TRUE;
            Done := FALSE;
            Error := FALSE;
            internalState := 10;
        END_IF
    
    10: (* Initiate connection *)
        extResult := Connect(
            ADR(ServerUrl), 
            TIME_TO_UDINT(Timeout),
            ADR(ErrorID));
        
        IF extResult <> 0 THEN
            internalState := 20; (* Success *)
        ELSE
            internalState := 30; (* Error *)
        END_IF
        
    20: (* Connection successful *)
        Done := TRUE;
        Busy := FALSE;
        internalState := 0;
        RETURN TRUE;
        
    30: (* Error handling *)
        Error := TRUE;
        Busy := FALSE;
        internalState := 0;
END_CASE

RETURN FALSE;
