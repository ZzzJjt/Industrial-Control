PROGRAM PLC_PRG
VAR
    // Batch Flags
    bStartReaction      : BOOL := FALSE;
    bReactionComplete   : BOOL := FALSE;
    bEmergencyStop      : BOOL := FALSE;

    // Process Parameters
    rSetTemperature     : REAL := 180.0; // °C
    rMaxTempTolerance   : REAL := 1.0;   // ±1°C tolerance
    rTargetPressure     : REAL := 140.0; // bar
    tHoldTime           : TIME := T#30m;
    rCoolDownTemp       : REAL := 80.0;  // °C

    // Equipment Status
    bHeaterOn           : BOOL := FALSE;
    bCoolerOn           : BOOL := FALSE;
    bPressureRegulating : BOOL := FALSE;

    // Sensors
    rCurrentTemp        : REAL := 0.0;
    rCurrentPressure    : REAL := 0.0;

    // Timers
    tmrHold             : TON;

    // State Machine
    eState              : E_UREA_REACTION_STEP := STEP_IDLE;
END_VAR

// Emergency Stop Override
IF bEmergencyStop THEN
    eState := STEP_ABORTED;
END_IF;

CASE eState OF
    STEP_IDLE:
        IF bStartReaction THEN
            StartPreheat(rSetTemperature);
            eState := STEP_PREHEAT_PHASE;
        END_IF;

    STEP_PREHEAT_PHASE:
        IF IsReactorHeated() THEN
            RegulatePressure(rTargetPressure);
            eState := STEP_PRESSURE_REGULATION;
        END_IF;

    STEP_PRESSURE_REGULATION:
        IF IsPressureStable() THEN
            StartHoldPhase(tHoldTime);
            eState := STEP_HOLD_PHASE;
        END_IF;

    STEP_HOLD_PHASE:
        IF IsHoldPhaseDone() THEN
            StartCoolDown(rCoolDownTemp);
            eState := STEP_COOL_DOWN_PHASE;
        END_IF;

    STEP_COOL_DOWN_PHASE:
        IF IsReactorCooled() THEN
            bReactionComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action required — handled by higher-level system

    STEP_ABORTED:
        AbortAllProcesses();
        bReactionComplete := FALSE;
END_CASE;

METHOD AbortAllProcesses
bHeaterOn := FALSE;
bCoolerOn := FALSE;
bPressureRegulating := FALSE;
tmrHold(IN := FALSE);
