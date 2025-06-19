PROGRAM PLC_PRG
VAR
    // State Machine
    eState              : E_REACTION_STEP := STEP_IDLE;
    bStartReaction      : BOOL := FALSE;
    bReactionComplete   : BOOL := FALSE;

    // Process Parameters
    rSetTemperature     : REAL := 85.0;  // °C
    rCurrentTemp        : REAL := 0.0;
    nMixSpeedRPM        : INT := 120;    // RPM
    tHoldTime           : TIME := T#30m;

    // Equipment Status
    bHeaterOn           : BOOL := FALSE;
    bMixerOn            : BOOL := FALSE;

    // Timers
    tmrHeatUp           : TON;
    tmrHold             : TON;

    // Interlocks & Safety
    bEmergencyStop      : BOOL := FALSE;
    bOverTempDetected   : BOOL := FALSE;
END_VAR

TYPE E_REACTION_STEP :
(
    STEP_IDLE,
    STEP_START_SEQUENCE,
    STEP_HEATING_PHASE,
    STEP_MIXING_PHASE,
    STEP_HOLD_PHASE,
    STEP_COMPLETE,
    STEP_ABORTED
);
END_TYPE

// Emergency Stop Override
IF bEmergencyStop THEN
    eState := STEP_ABORTED;
END_IF;

CASE eState OF
    STEP_IDLE:
        IF bStartReaction THEN
            eState := STEP_START_SEQUENCE;
        END_IF;

    STEP_START_SEQUENCE:
        StartHeating(rSetTemperature);
        StartMixing(nMixSpeedRPM);
        eState := STEP_HEATING_PHASE;

    STEP_HEATING_PHASE:
        IF HeaterAtTargetTemp(rSetTemperature) THEN
            tmrHold(IN := TRUE, PT := tHoldTime); // Begin hold phase timer
            eState := STEP_HOLD_PHASE;
        END_IF;

    STEP_HOLD_PHASE:
        IF tmrHold.Q THEN
            StopMixing();
            bReactionComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action required; handled by higher-level batch controller

    STEP_ABORTED:
        StopAllSystems();
        bReactionComplete := FALSE;
END_CASE;

METHOD StartHeating
VAR_INPUT setTemp: REAL; END_VAR
bHeaterOn := TRUE;
rSetTemperature := setTemp;
tmrHeatUp(IN := TRUE, PT := T#15m); // Max allowed heat-up time
