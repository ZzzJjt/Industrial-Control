PROGRAM PLC_PRG
VAR
    // Batch Flags
    bStartBatch         : BOOL := FALSE;
    bBatchComplete      : BOOL := FALSE;
    bEmergencyStop      : BOOL := FALSE;

    // Process Parameters
    rReactorTempMin     : REAL := 55.0;  // °C
    rReactorTempMax     : REAL := 60.0;
    rTargetPressure     : REAL := 0.2;   // bar (indicating completion)
    tReactionTime       : TIME := T#4h;
    rWaterVolume        : REAL := 500.0; // L
    tDryTime            : TIME := T#2h;

    // Equipment Status
    bEvacuationActive   : BOOL := FALSE;
    bChargingActive     : BOOL := FALSE;
    bPolymerizing       : BOOL := FALSE;
    bDecovering         : BOOL := FALSE;
    bDrying             : BOOL := FALSE;

    // Sensors
    rReactorTemp        : REAL := 0.0;
    rReactorPressure    : REAL := 0.0;
    rWaterAdded         : REAL := 0.0;

    // Timers
    tmrReaction         : TON;
    tmrDrying           : TON;

    // State Machine
    eState              : E_PVC_BATCH_STEP := STEP_IDLE;
END_VAR

// Emergency Stop Override
IF bEmergencyStop THEN
    eState := STEP_ABORTED;
END_IF;

CASE eState OF
    STEP_IDLE:
        IF bStartBatch THEN
            StartEvacuation();
            eState := STEP_EVACUATE_REACTOR;
        END_IF;

    STEP_EVACUATE_REACTOR:
        IF IsEvacuationDone() THEN
            AddDemineralizedWater(rWaterVolume);
            eState := STEP_CHARGE_WATER_SURFACTANT;
        END_IF;

    STEP_CHARGE_WATER_SURFACTANT:
        IF IsWaterAndSurfactantAdded() THEN
            StartPolymerization(rReactorTempMin, rReactorTempMax, tReactionTime);
            eState := STEP_POLYMERIZATION_PHASE;
        END_IF;

    STEP_POLYMERIZATION_PHASE:
        IF IsPolymerizationComplete() THEN
            StartDecover();
            eState := STEP_DECOVER_PHASE;
        END_IF;

    STEP_DECOVER_PHASE:
        Delay(T#10m); // Simulate decanter time or venting
        StartDrying(tDryTime);
        eState := STEP_DRYING_PHASE;

    STEP_DRYING_PHASE:
        IF IsDryingDone() THEN
            bBatchComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action required — handled by higher-level system

    STEP_ABORTED:
        AbortAllProcesses();
        bBatchComplete := FALSE;
END_CASE;
