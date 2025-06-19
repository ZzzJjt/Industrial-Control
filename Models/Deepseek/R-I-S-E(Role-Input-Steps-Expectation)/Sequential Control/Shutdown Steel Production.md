PROGRAM PLC_PRG
VAR
    // Shutdown Flags
    bStartShutdown      : BOOL := FALSE;
    bShutdownComplete   : BOOL := FALSE;
    bEmergencyStop      : BOOL := FALSE;

    // Process Parameters
    rTargetTemp         : REAL := 400.0;     // °C
    tCoolingTime        : TIME := T#8h;      // Time to reach target temp
    tGasRampDownTime    : TIME := T#12h;     // Duration of fuel gas reduction
    rFuelToAirRatio     : REAL := 2.5;       // Desired air/fuel ratio

    // Equipment Status
    bFurnaceCooling     : BOOL := FALSE;
    bGasFlowRamping     : BOOL := FALSE;
    bOxygenRegulating   : BOOL := FALSE;

    // Sensors
    rCurrentTemp        : REAL := 0.0;       // Current furnace temp
    rCurrentFuelFlow    : REAL := 0.0;       // Fuel flow rate (e.g., m³/hr)
    rCurrentOxygenLevel : REAL := 0.0;       // % O₂ in combustion air

    // Timers
    tmrCooling          : TON;
    tmrGasRampDown      : TON;

    // State Machine
    eState              : E_SHUTDOWN_STEP := STEP_IDLE;

    // Outputs
    rGasValveOutput     : REAL := 0.0;       // % valve opening
    rAirDamperOutput    : REAL := 0.0;       // % damper position
    bAlarmHighTemp      : BOOL := FALSE;
    bAlarmLowOxygen     : BOOL := FALSE;
END_VAR

// Emergency Stop Override
IF bEmergencyStop THEN
    eState := STEP_ABORTED;
END_IF;

CASE eState OF
    STEP_IDLE:
        IF bStartShutdown THEN
            StartFurnaceCooling(rTargetTemp, tCoolingTime);
            eState := STEP_FURNACE_COOLING;
        END_IF;

    STEP_FURNACE_COOLING:
        IF IsFurnaceCooled() THEN
            StartGasFlowRampDown(tGasRampDownTime);
            eState := STEP_GAS_FLOW_RAMP_DOWN;
        END_IF;

    STEP_GAS_FLOW_RAMP_DOWN:
        IF IsGasRampDone() THEN
            StartOxygenRegulation(rFuelToAirRatio);
            eState := STEP_OXYGEN_REGULATION;
        END_IF;

    STEP_OXYGEN_REGULATION:
        IF IsOxygenStable() THEN
            bShutdownComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action required — handled by higher-level system

    STEP_ABORTED:
        AbortAllProcesses();
        bShutdownComplete := FALSE;
END_CASE;
