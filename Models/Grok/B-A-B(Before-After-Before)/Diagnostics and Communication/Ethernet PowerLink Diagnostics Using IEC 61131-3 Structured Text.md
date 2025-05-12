(* Function Block: POWERLINK_DIAGNOSTICS
   Purpose: Retrieves diagnostic data from Ethernet PowerLink Managing Node (MN) for connected control nodes.
   Features:
   - Cyclically polls MN for node status, error codes, and health
   - Provides structured outputs for node communication status and faults
   - Includes error handling for diagnostic channel issues
   - Supports up to 32 nodes for scalability
*)

FUNCTION_BLOCK POWERLINK_DIAGNOSTICS
VAR_INPUT
    ENABLE : BOOL;                (* Enables/disables diagnostic polling *)
    POLL_INTERVAL : TIME;         (* Polling interval, e.g., T#500ms *)
    MAX_NODES : UINT := 32;       (* Maximum number of nodes to monitor *)
END_VAR

VAR_OUTPUT
    DONE : BOOL;                  (* TRUE when polling cycle completes successfully *)
    BUSY : BOOL;                  (* TRUE during polling *)
    ERROR : BOOL;                 (* TRUE if an error occurs *)
    ERROR_CODE : UINT;            (* 0=None, 1=Comm Failure, 2=Invalid Node, 3=Timeout *)
    NODE_STATUS : ARRAY[0..31] OF UINT; (* Node status: 0=Offline, 1=Operational, 2=Error *)
    NODE_ERROR_CODE : ARRAY[0..31] OF UINT; (* Node error: 0=None, 1=Timeout, 2=Frame Error, 3=Internal Fault *)
    NODE_HEALTH : ARRAY[0..31] OF BOOL; (* TRUE if node is healthy (Operational, no errors) *)
END_VAR

VAR
    Timer : TON;                  (* Timer for polling interval *)
    CurrentNode : UINT;           (* Current node being polled *)
    PollComplete : BOOL;          (* Flag for completed polling cycle *)
    MnResponse : UINT;            (* Response from MN diagnostic channel *)
    LastEnable : BOOL;            (* Tracks previous ENABLE state *)
    i : UINT;                     (* Loop index *)
END_VAR

(* Initialize outputs *)
DONE := FALSE;
BUSY := FALSE;
ERROR := FALSE;
ERROR_CODE := 0;
FOR i := 0 TO MAX_NODES - 1 DO
    NODE_STATUS[i] := 0;
    NODE_ERROR_CODE[i] := 0;
    NODE_HEALTH[i] := FALSE;
END_FOR

(* Main logic *)
IF ENABLE THEN
    (* Initialize on rising edge of ENABLE *)
    IF NOT LastEnable THEN
        Timer(IN := FALSE);
        CurrentNode := 0;
        PollComplete := FALSE;
        BUSY := FALSE;
        DONE := FALSE;
        ERROR := FALSE;
        ERROR_CODE := 0;
        FOR i := 0 TO MAX_NODES - 1 DO
            NODE_STATUS[i] := 0;
            NODE_ERROR_CODE[i] := 0;
            NODE_HEALTH[i] := FALSE;
        END_FOR
    END_IF
    
    (* Start polling timer *)
    Timer(IN := TRUE, PT := POLL_INTERVAL);
    
    (* Poll when timer elapses *)
    IF Timer.Q AND NOT PollComplete THEN
        BUSY := TRUE;
        DONE := FALSE;
        
        (* Validate node index *)
        IF CurrentNode >= MAX_NODES THEN
            ERROR := TRUE;
            ERROR_CODE := 2; (* Invalid Node *)
            BUSY := FALSE;
            DONE := FALSE;
            Timer(IN := FALSE);
        ELSE
            (* Poll diagnostic data for current node *)
            MnResponse := PLK_GetNodeDiagnostics(CurrentNode);
            
            (* Process response *)
            CASE MnResponse OF
                0: (* Node Offline *)
                    NODE_STATUS[CurrentNode] := 0;
                    NODE_ERROR_CODE[CurrentNode] := 0;
                    NODE_HEALTH[CurrentNode] := FALSE;
                1: (* Node Operational *)
                    NODE_STATUS[CurrentNode] := 1;
                    NODE_ERROR_CODE[CurrentNode] := 0;
                    NODE_HEALTH[CurrentNode] := TRUE;
                2: (* Node Error - Timeout *)
                    NODE_STATUS[CurrentNode] := 2;
                    NODE_ERROR_CODE[CurrentNode] := 1;
                    NODE_HEALTH[CurrentNode] := FALSE;
                3: (* Node Error - Frame Error *)
                    NODE_STATUS[CurrentNode] := 2;
                    NODE_ERROR_CODE[CurrentNode] := 2;
                    NODE_HEALTH[CurrentNode] := FALSE;
                4: (* Node Error - Internal Fault *)
                    NODE_STATUS[CurrentNode] := 2;
                    NODE_ERROR_CODE[CurrentNode] := 3;
                    NODE_HEALTH[CurrentNode] := FALSE;
                100: (* Communication Failure *)
                    ERROR := TRUE;
                    ERROR_CODE := 1; (* Comm Failure *)
                    BUSY := FALSE;
                    DONE := FALSE;
                    Timer(IN := FALSE);
                    RETURN;
                101: (* Timeout *)
                    ERROR := TRUE;
                    ERROR_CODE := 3; (* Timeout *)
                    BUSY := FALSE;
                    DONE := FALSE;
                    Timer(IN := FALSE);
                    RETURN;
                ELSE
                    (* Unknown response *)
                    ERROR := TRUE;
                    ERROR_CODE := 1; (* Comm Failure *)
                    BUSY := FALSE;
                    DONE := FALSE;
                    Timer(IN := FALSE);
                    RETURN;
            END_CASE
            
            (* Move to next node *)
            CurrentNode := CurrentNode + 1;
            
            (* Check if all nodes polled *)
            IF CurrentNode >= MAX_NODES THEN
                PollComplete := TRUE;
                CurrentNode := 0;
                DONE := TRUE;
                BUSY := FALSE;
                Timer(IN := FALSE);
            END_IF
        END_IF
    END_IF
    
    (* Reset PollComplete for next cycle *)
    IF PollComplete AND Timer.Q THEN
        PollComplete := FALSE;
        DONE := FALSE;
    END_IF
ELSE
    (* Disable monitoring *)
    Timer(IN := FALSE);
    BUSY := FALSE;
    DONE := FALSE;
    ERROR := FALSE;
    ERROR_CODE := 0;
    CurrentNode := 0;
    PollComplete := FALSE;
    FOR i := 0 TO MAX_NODES - 1 DO
        NODE_STATUS[i] := 0;
        NODE_ERROR_CODE[i] := 0;
        NODE_HEALTH[i] := FALSE;
    END_FOR
END_IF

(* Update edge detection *)
LastEnable := ENABLE;

(* Simulated PowerLink diagnostic function *)
FUNCTION PLK_GetNodeDiagnostics : UINT
VAR_INPUT
    NodeId : UINT;
END_VAR
(* Placeholder: Simulates MN diagnostic response *)
(* Returns: 0=Offline, 1=Operational, 2=Timeout, 3=Frame Error, 4=Internal Fault, 100=Comm Failure, 101=Timeout *)
PLK_GetNodeDiagnostics := 1; (* Simulate Operational *)
END_FUNCTION
END_FUNCTION_BLOCK
