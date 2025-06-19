PROGRAM PLC_PRG
VAR
    // Batch Flags
    bStartBatch         : BOOL := FALSE;
    bBatchComplete      : BOOL := FALSE;
    bEmergencyStop      : BOOL := FALSE;

    // Ingredients
    rMilkAdded          : REAL := 0.0;  // kg
    rWaterAdded         : REAL := 0.0;
    rSugarAdded         : REAL := 0.0;
    rCocoaAdded         : REAL := 0.0;

    // Target Quantities
    rMilkTarget         : REAL := 60.0;
    rWaterTarget        : REAL := 20.0;
    rSugarTarget        : REAL := 15.0;
    rCocoaTarget        : REAL := 5.0;

    // Process Parameters
    rSetTemperature     : REAL := 70.0;  // °C
    nMixSpeedRPM        : INT := 200;
    tMixTime            : TIME := T#10m;

    // Equipment Status
    bMilkValveOpen      : BOOL := FALSE;
    bWaterValveOpen     : BOOL := FALSE;
    bSugarValveOpen     : BOOL := FALSE;
    bCocoaDispenserOn   : BOOL := FALSE;
    bHeaterOn           : BOOL := FALSE;
    bMixerOn            : BOOL := FALSE;

    // Sensors
    rCurrentTemp        : REAL := 0.0;

    // Timers
    tmrHeatUp           : TON;
    tmrMix              : TON;

    // State Machine
    eState              : E_COCOA_MILK_STEP := STEP_IDLE;
END_VAR

TYPE E_COCOA_MILK_STEP :
(
    STEP_IDLE,
    STEP_ADD_MILK,
    STEP_ADD_WATER,
    STEP_ADD_SUGAR,
    STEP_ADD_COCOA,
    STEP_HEATING_PHASE,
    STEP_MIXING_PHASE,
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
            StartAddingMilk();
            eState := STEP_ADD_MILK;
        END_IF;

    STEP_ADD_MILK:
        IF IsMilkAdded() THEN
            StartAddingWater();
            eState := STEP_ADD_WATER;
        END_IF;

    STEP_ADD_WATER:
        IF IsWaterAdded() THEN
            StartAddingSugar();
            eState := STEP_ADD_SUGAR;
        END_IF;

    STEP_ADD_SUGAR:
        IF IsSugarAdded() THEN
            StartAddingCocoa();
            eState := STEP_ADD_COCOA;
        END_IF;

    STEP_ADD_COCOA:
        IF IsCocoaAdded() THEN
            StartHeating(rSetTemperature);
            eState := STEP_HEATING_PHASE;
        END_IF;

    STEP_HEATING_PHASE:
        IF IsHeatedToTarget() THEN
            StartMixing(nMixSpeedRPM, tMixTime);
            eState := STEP_MIXING_PHASE;
        END_IF;

    STEP_MIXING_PHASE:
        IF IsMixingDone() THEN
            bBatchComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action required — handled by higher-level system

    STEP_ABORTED:
        AbortAllProcesses();
        bBatchComplete := FALSE;
END_CASE;

FUNCTION_BLOCK IsMilkAdded RETURNS BOOL
IF rMilkAdded >= rMilkTarget THEN
    RETURN := TRUE;
ELSE
    RETURN := FALSE;
END_IF;
