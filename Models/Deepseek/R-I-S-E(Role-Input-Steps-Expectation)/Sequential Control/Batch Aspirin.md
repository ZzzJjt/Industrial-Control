PROGRAM PLC_PRG
VAR
    // Batch Control Flags
    bStartBatch         : BOOL := FALSE;
    bBatchComplete      : BOOL := FALSE;
    bEmergencyStop      : BOOL := FALSE;

    // Process Parameters
    rReactionTemp       : REAL := 85.0;  // °C
    tReactionTime       : TIME := T#30m;
    rCrystallizationTemp: REAL := 25.0;  // °C
    tCrystallizationTime: TIME := T#1h;
    tDryerTemp          : REAL := 90.0;  // °C
    tDryingTime         : TIME := T#2h;

    // Equipment Status
    bReactorHeaterOn    : BOOL := FALSE;
    bMixerOn            : BOOL := FALSE;
    bCrystallizerActive : BOOL := FALSE;
    bCentrifugeOn       : BOOL := FALSE;
    bDryerOn            : BOOL := FALSE;

    // Sensors
    rReactorTemp        : REAL := 0.0;
    rReactorPressure    : REAL := 0.0;
    rDryerTemp          : REAL := 0.0;

    // Timers
    tmrReaction         : TON;
    tmrCrystallization  : TON;
    tmrDrying           : TON;

    // State Machine
    eState              : E_ASPRIN_BATCH_STEP := STEP_IDLE;
END_VAR

TYPE E_ASPRIN_BATCH_STEP :
(
    STEP_IDLE,
    STEP_REACTION_PHASE,
    STEP_CRYSTALLIZATION_PHASE,
    STEP_CENTRIFUGE_PHASE,
    STEP_DRYING_PHASE,
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
        IF bStartBatch THEN
            StartReaction(rReactionTemp, tReactionTime);
            eState := STEP_REACTION_PHASE;
        END_IF;

    STEP_REACTION_PHASE:
        IF IsReactionComplete() THEN
            StartCrystallization(rCrystallizationTemp, tCrystallizationTime);
            eState := STEP_CRYSTALLIZATION_PHASE;
        END_IF;

    STEP_CRYSTALLIZATION_PHASE:
        IF IsCrystallizationComplete() THEN
            StartCentrifuge();
            eState := STEP_CENTRIFUGE_PHASE;
        END_IF;

    STEP_CENTRIFUGE_PHASE:
        Delay(T#5m); // Simulate centrifuge time
        StartDrying(tDryingTime);
        eState := STEP_DRYING_PHASE;

    STEP_DRYING_PHASE:
        IF IsDryingComplete() THEN
            bBatchComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action needed — handled by higher-level system

    STEP_ABORTED:
        AbortAllProcesses();
        bBatchComplete := FALSE;
END_CASE;
