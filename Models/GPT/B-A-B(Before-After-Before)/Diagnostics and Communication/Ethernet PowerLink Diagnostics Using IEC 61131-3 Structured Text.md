FUNCTION_BLOCK EPL_DIAG_MONITOR
VAR_INPUT
    ENABLE       : BOOL;             // Starts diagnostic polling
    NUM_NODES    : INT;              // Number of nodes to monitor (1..10)
END_VAR

VAR_OUTPUT
    NodeStatus   : ARRAY[1..10] OF BOOL;     // TRUE if node is online
    NodeErrors   : ARRAY[1..10] OF INT;      // Error code for each node
    NodeHealth   : ARRAY[1..10] OF STRING[20]; // 'OK', 'Timeout', 'Offline', 'InternalErr', etc.
    NewStatus    : BOOL;                     // TRUE if new diagnostic info is available
END_VAR

VAR
    CurrentNode  : INT := 1;
    DiagRequest  : BOOL := FALSE;
    DiagBusy     : BOOL := FALSE;
    DiagDone     : BOOL := FALSE;
    DiagError    : BOOL := FALSE;
    DiagCode     : INT;
    DiagStatus   : INT;
    PollTimer    : TON;
    Triggered    : BOOL := FALSE;
END_VAR

// Simulated diagnostic request block call (replace with vendor-specific EPL diagnostic service)
FUNCTION_BLOCK READ_DIAG
VAR_INPUT
    NODE_ID : INT;
    ENABLE  : BOOL;
END_VAR
VAR_OUTPUT
    DONE    : BOOL;
    ERROR   : BOOL;
    STATUS  : INT;  // 0 = OK, 1 = Timeout, 2 = Offline, 3 = InternalErr
    CODE    : INT;  // Specific error code
END_VAR

// Instantiate simulated READ_DIAG for node communication
VAR
    ReadDiagFB : READ_DIAG;
END_VAR

// Polling timer (e.g., every 500 ms)
PollTimer(IN := ENABLE, PT := T#500ms);

// Trigger once per cycle
IF PollTimer.Q AND ENABLE THEN
    IF NOT Triggered THEN
        Triggered := TRUE;
        NewStatus := FALSE;

        // Trigger READ_DIAG for current node
        ReadDiagFB.NODE_ID := CurrentNode;
        ReadDiagFB.ENABLE := TRUE;
    END_IF
ELSE
    Triggered := FALSE;
END_IF

// Process READ_DIAG results
IF ReadDiagFB.DONE THEN
    DiagStatus := ReadDiagFB.STATUS;
    DiagCode := ReadDiagFB.CODE;

    CASE DiagStatus OF
        0:
            NodeStatus[CurrentNode] := TRUE;
            NodeErrors[CurrentNode] := 0;
            NodeHealth[CurrentNode] := 'OK';
        1:
            NodeStatus[CurrentNode] := FALSE;
            NodeErrors[CurrentNode] := DiagCode;
            NodeHealth[CurrentNode] := 'Timeout';
        2:
            NodeStatus[CurrentNode] := FALSE;
            NodeErrors[CurrentNode] := DiagCode;
            NodeHealth[CurrentNode] := 'Offline';
        3:
            NodeStatus[CurrentNode] := FALSE;
            NodeErrors[CurrentNode] := DiagCode;
            NodeHealth[CurrentNode] := 'InternalErr';
        ELSE
            NodeStatus[CurrentNode] := FALSE;
            NodeErrors[CurrentNode] := -1;
            NodeHealth[CurrentNode] := 'Unknown';
    END_CASE

    ReadDiagFB.ENABLE := FALSE;
    NewStatus := TRUE;

    // Next node (circular polling)
    IF CurrentNode < NUM_NODES THEN
        CurrentNode := CurrentNode + 1;
    ELSE
        CurrentNode := 1;
    END_IF

ELSIF ReadDiagFB.ERROR THEN
    NodeStatus[CurrentNode] := FALSE;
    NodeErrors[CurrentNode] := -999;
    NodeHealth[CurrentNode] := 'DiagCommErr';

    ReadDiagFB.ENABLE := FALSE;
    NewStatus := TRUE;

    // Proceed to next node despite error
    IF CurrentNode < NUM_NODES THEN
        CurrentNode := CurrentNode + 1;
    ELSE
        CurrentNode := 1;
    END_IF
END_IF
