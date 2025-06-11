PROGRAM PLC_PRG
VAR
    // Batch Flags
    bStartBatch         : BOOL := FALSE;
    bBatchComplete      : BOOL := FALSE;
    bEmergencyStop      : BOOL := FALSE;

    // Process Parameters
    rEthyleneAmount     : REAL := 1000.0; // kg
    rCatalystAmount     : REAL := 5.0;    // kg
    rAdditiveAmount     : REAL := 20.0;   // kg
    rPolymerTemp        : REAL := 90.0;   // °C
    tPolymerTime        : TIME := T#3h;
    rDryerTemp          : REAL := 70.0;   // °C
    tDryingTime         : TIME := T#2h;

    // Equipment Status
    bRawMaterialPrepared: BOOL := FALSE;
    bPolymerizing       : BOOL := FALSE;
    bQuenching          : BOOL := FALSE;
    bDrying             : BOOL := FALSE;
    bPelletizing        : BOOL := FALSE;
    bQCCompleted        : BOOL := FALSE;
    bPackaging          : BOOL := FALSE;

    // Sensors
    rCurrentTemp        : REAL := 0.0;
    rCurrentPressure    : REAL := 0.0;

    // Timers
    tmrPolymerization   : TON;
    tmrDrying           : TON;

    // State Machine
    eState              : E_PE_BATCH_STEP := STEP_IDLE;
END_VAR

// Emergency Stop Override
IF bEmergencyStop THEN
    eState := STEP_ABORTED;
END_IF;

CASE eState OF
    STEP_IDLE:
        IF bStartBatch THEN
            StartRawMaterialPreparation(rEthyleneAmount, rCatalystAmount, rAdditiveAmount);
            eState := STEP_RAW_MATERIAL_PREPARATION;
        END_IF;

    STEP_RAW_MATERIAL_PREPARATION:
        IF IsRawMaterialPrepared() THEN
            StartPolymerization(rPolymerTemp, tPolymerTime);
            eState := STEP_POLYMERIZATION_PHASE;
        END_IF;

    STEP_POLYMERIZATION_PHASE:
        IF IsPolymerizationComplete() THEN
            StartQuenching();
            eState := STEP_QUENCHING_PHASE;
        END_IF;

    STEP_QUENCHING_PHASE:
        Delay(T#10m); // Simulate quenching time
        StartDrying(rDryerTemp, tDryingTime);
        eState := STEP_DRYING_PHASE;

    STEP_DRYING_PHASE:
        IF IsDryingDone() THEN
            StartPelletizing();
            eState := STEP_PELLETIZING_PHASE;
        END_IF;

    STEP_PELLETIZING_PHASE:
        Delay(T#10m); // Simulate pelletizing time
        PerformQualityControl();
        eState := STEP_QUALITY_CONTROL;

    STEP_QUALITY_CONTROL:
        IF IsQCCompleted() THEN
            StartPackaging();
            eState := STEP_PACKAGING_AND_STORAGE;
        END_IF;

    STEP_PACKAGING_AND_STORAGE:
        IF IsPackagingDone() THEN
            bBatchComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action required — handled by higher-level system

    STEP_ABORTED:
        AbortAllProcesses();
        bBatchComplete := FALSE;
END_CASE;
