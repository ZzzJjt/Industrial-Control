PROGRAM PLC_PRG
VAR
    // Batch process states
    eBatchState : E_BATCH_STATE := E_BATCH_STATE_IDLE;

    // Timers for each step
    tmrStepTimer : TON;

    // Flags for step initialization
    bStepInitialized : BOOL := FALSE;

    // Process parameters (example values)
    rTargetTemp : REAL := 0.0;
    rTargetPressure : REAL := 0.0;
END_VAR

CASE eBatchState OF
    E_BATCH_STATE_IDLE:
        IF StartBatch THEN
            eBatchState := E_BATCH_STATE_RAW_MATERIAL_PREP;
            bStepInitialized := FALSE;
        END_IF;

    E_BATCH_STATE_RAW_MATERIAL_PREP:
        StepControl(duration := T#10s, nextState := E_BATCH_STATE_POLYMERIZATION);

    E_BATCH_STATE_POLYMERIZATION:
        StepControl(duration := T#30s, nextState := E_BATCH_STATE_QUENCHING);

    E_BATCH_STATE_QUENCHING:
        StepControl(duration := T#15s, nextState := E_BATCH_STATE_DRYING);

    E_BATCH_STATE_DRYING:
        StepControl(duration := T#20s, nextState := E_BATCH_STATE_PELLETIZING);

    E_BATCH_STATE_PELLETIZING:
        StepControl(duration := T#25s, nextState := E_BATCH_STATE_QA);

    E_BATCH_STATE_QA:
        StepControl(duration := T#10s, nextState := E_BATCH_STATE_PACKAGING);

    E_BATCH_STATE_PACKAGING:
        StepControl(duration := T#15s, nextState := E_BATCH_STATE_COMPLETE [ ](c) 


    CASE eBatchState OF
    E_BATCH_STATE_RAW_MATERIAL_PREP:
        rTargetTemp := 80.0;     // °C
        rTargetPressure := 1.0;  // bar

    E_BATCH_STATE_POLYMERIZATION:
        rTargetTemp := 220.0;
        rTargetPressure := 30.0;

    E_BATCH_STATE_QUENCHING:
        rTargetTemp := 40.0;
        rTargetPressure := 1.5;

    E_BATCH_STATE_DRYING:
        rTargetTemp := 100.0;
        rTargetPressure := 1.2;

    E_BATCH_STATE_PELLETIZING:
        rTargetTemp := 180.0;
        rTargetPressure := 20.0;

    E_BATCH_STATE_QA, E_BATCH_STATE_PACKAGING:
        rTargetTemp := 25.0;
        rTargetPressure := 1.0;

    ELSE
        rTargetTemp := 0.0;
        rTargetPressure := 0.0;
END_CASE;

// Apply conditions to physical system here (e.g., write to PID setpoints)
ApplyTemperature(rTargetTemp);
ApplyPressure(rTargetPressure);
