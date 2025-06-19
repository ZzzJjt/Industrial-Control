TYPE POWERLINK_NODE_STATE :
(
    OFFLINE = 0,
    ONLINE = 1,
    FAULTED = 2,
    INITIALIZING = 3
);
END_TYPE

TYPE POWERLINK_DIAGNOSTICS:
STRUCT
    NodeID: UINT;                      // PowerLink node ID
    State: POWERLINK_NODE_STATE;       // Current node state
    LastErrorCode: UDINT;              // Last recorded error code
    CommunicationUp: BOOL;             // Communication status
    FrameLossCount: UDINT;             // Number of lost frames
    CRCErrorCount: UDINT;              // CRC errors in received frames
    HealthScore: BYTE;                 // Node health estimate (0–100%)
    LastUpdate: DATE_AND_TIME;         // Timestamp of last update
END_STRUCT
END_TYPE

FUNCTION_BLOCK FB_PowerLinkDiagnostics
VAR_INPUT
    NODE_ID: UINT := 0;                // Node to query (0 = broadcast or all nodes)
    TRIGGER_POLL: BOOL := FALSE;       // TRUE = trigger new poll cycle
    ENABLE: BOOL := TRUE;              // Enable cyclic polling
    POLL_INTERVAL: TIME := T#5s;       // Default polling interval
END_VAR

VAR_OUTPUT
    DIAG_DATA: POWERLINK_DIAGNOSTICS;  // Latest diagnostic data
    COMM_FAILURE: BOOL := FALSE;       // General communication failure flag
    LAST_ERROR_CODE: UDINT := 0;       // Specific error code
    UPDATED: BOOL := FALSE;            // Indicates new data available
END_VAR

VAR
    pollTimer: TON;                     // Timer for polling interval
    prevTrigger: BOOL := FALSE;
    tempDiagData: POWERLINK_DIAGNOSTICS;
END_VAR

// Reset updated flag at start of scan
UPDATED := FALSE;

// Only run if enabled
IF NOT ENABLE THEN
    RETURN;
END_IF;

// Start timer for cyclic polling
pollTimer(IN := TRUE, PT := POLL_INTERVAL);

// Trigger on rising edge of manual trigger or timer expiration
IF (TRIGGER_POLL AND NOT prevTrigger) OR pollTimer.Q THEN
    prevTrigger := TRIGGER_POLL;

    // Attempt to retrieve diagnostics
    IF PL_GetNodeDiagnostics(NODE_ID, ADR(tempDiagData)) THEN
        // Successfully retrieved data
        DIAG_DATA := tempDiagData;
        DIAG_DATA.LastUpdate := CURRENT_DATE_TIME();
        UPDATED := TRUE;
        COMM_FAILURE := FALSE;
        LAST_ERROR_CODE := 0;

    ELSE
        // Communication failed
        COMM_FAILURE := TRUE;
        LAST_ERROR_CODE := 1; // e.g., timeout or invalid response
        // Optionally log or store error history here
    END_IF;

    // Reset timer after action
    pollTimer(IN := FALSE);
    pollTimer(IN := TRUE); // Restart timer
END_IF;

METHOD PRIVATE CalculateHealthScore : BYTE
VAR_INPUT
    crcErrors: UDINT;
    frameLoss: UDINT;
    maxErrors: UDINT := 1000;
END_VAR

VAR
    totalErrors: UDINT;
    healthPercent: REAL;
END_VAR

totalErrors := crcErrors + frameLoss;
healthPercent := MAX(0.0, 100.0 - (REAL_TO_REAL(totalErrors) / REAL_TO_REAL(maxErrors)) * 100.0);
CalculateHealthScore := REAL_TO_BYTE(healthPercent);

PROGRAM PLC_PRG
VAR
    powerLinkMon: FB_PowerLinkDiagnostics;
    nodeIDToMonitor: UINT := 5;
    diagnosticsReady: BOOL := FALSE;
    latestDiag: POWERLINK_DIAGNOSTICS;
END_VAR

powerLinkMon(
    NODE_ID := nodeIDToMonitor,
    TRIGGER_POLL := FALSE,     // Use cyclic polling
    ENABLE := TRUE,
    POLL_INTERVAL := T#5s
);

// Output results
latestDiag := powerLinkMon.DIAG_DATA;
diagnosticsReady := powerLinkMon.UPDATED;

// HMI or alarm integration
IF powerLinkMon.COMM_FAILURE THEN
    hmiAlarmText := CONCAT('PowerLink Comm Failure with Node ', UINT_TO_STRING(nodeIDToMonitor));
    hmiAlarmActive := TRUE;
END_IF;
