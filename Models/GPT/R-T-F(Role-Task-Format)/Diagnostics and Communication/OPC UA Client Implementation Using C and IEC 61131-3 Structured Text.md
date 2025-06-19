#ifndef OPCUA_CLIENT_H
#define OPCUA_CLIENT_H

#include <stdint.h>
#include <stdbool.h>

#ifdef __cplusplus
extern "C" {
#endif

typedef struct {
    bool done;
    bool busy;
    bool error;
    uint32_t errorId;
} OPCUA_ClientStatus;

void OPCUA_ClientInit(void);
void OPCUA_ClientConnect(const char* serverUrl, uint32_t timeoutMs, OPCUA_ClientStatus* status);

#ifdef __cplusplus
}
#endif

#endif

FUNCTION_BLOCK FB_OPCUA_Connect
VAR_INPUT
    Execute     : BOOL;
    ServerUrl   : STRING(255);
    Timeout     : TIME;
END_VAR

VAR_OUTPUT
    Done        : BOOL;
    Busy        : BOOL;
    Error       : BOOL;
    ErrorID     : DWORD;
END_VAR

VAR
    RisingEdge  : R_TRIG;
    CallActive  : BOOL := FALSE;
    Status      : OPCUA_ClientStatus; // Imported C-struct
END_VAR

(* Rising edge detection for Execute *)
RisingEdge(CLK := Execute);

IF RisingEdge.Q THEN
    Busy := TRUE;
    Done := FALSE;
    Error := FALSE;
    ErrorID := 0;
    CallActive := TRUE;
    
    (* Call the C OPC UA client connect function *)
    OPCUA_ClientInit(); // Resets client
    OPCUA_ClientConnect(ServerUrl := ServerUrl, TimeoutMs := TO_DINT(Timeout) / 1000, Status := ADR(Status));
END_IF

IF CallActive THEN
    Busy := Status.busy;
    Done := Status.done;
    Error := Status.error;
    ErrorID := Status.errorId;

    IF Done OR Error THEN
        CallActive := FALSE;
    END_IF
END_IF
