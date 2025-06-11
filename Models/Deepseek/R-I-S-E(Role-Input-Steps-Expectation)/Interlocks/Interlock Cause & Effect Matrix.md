FUNCTION_BLOCK FB_ReactorInterlock
VAR_INPUT
    // Input Signals (Causes)
    bPressureHigh: BOOL;           // C01
    bTempHigh: BOOL;               // C02
    bLevelHigh: BOOL;              // C03
    bLevelLow: BOOL;               // C04
    bAgitatorFailure: BOOL;        // C05
    bFeedFlowLoss: BOOL;           // C06
    bCoolingWaterLoss: BOOL;       // C07
    bPHDeviation: BOOL;            // C08
    bGasLeakDetected: BOOL;        // C09
    bEmergencyStop: BOOL;          // C10
    bFlameFailure: BOOL;           // C11
    bPowerFailure: BOOL;           // C12
    bOverfill: BOOL;               // C13
    bSensorFault: BOOL;            // C14
END_VAR

VAR_OUTPUT
    // Output Actions (Effects)
    vFeedValve_Close: BOOL;                // E01
    vReliefValve_Open: BOOL;               // E02
    bEmergencyCooling_ON: BOOL;            // E03
    bAgitator_Stop: BOOL;                  // E04
    bFeedPump_Stop: BOOL;                  // E05
    bESDTriggered: BOOL;                    // E06
    bLocalAlarm_Sound: BOOL;               // E07
    bSCADA_Alarm_Send: BOOL;               // E08
    bLockoutAllInputs: BOOL;               // E09
    bIsolateReactor: BOOL;                  // E10
    bVentGasSafely: BOOL;                   // E11
    bSwitchToBackupPower: BOOL;            // E12
    bLogEvent: BOOL;                        // E13
    bFireSuppression_ON: BOOL;             // E14
    bControlledDepressurization: BOOL;     // E15
END_VAR

VAR
    bAnyCriticalFault: BOOL;
END_VAR

// Reset all outputs at start of scan
vFeedValve_Close := FALSE;
vReliefValve_Open := FALSE;
bEmergencyCooling_ON := FALSE;
bAgitator_Stop := FALSE;
bFeedPump_Stop := FALSE;
bESDTriggered := FALSE;
bLocalAlarm_Sound := FALSE;
bSCADA_Alarm_Send := FALSE;
bLockoutAllInputs := FALSE;
bIsolateReactor := FALSE;
bVentGasSafely := FALSE;
bSwitchToBackupPower := FALSE;
bLogEvent := FALSE;
bFireSuppression_ON := FALSE;
bControlledDepressurization := FALSE;

// Detect if any critical condition exists
bAnyCriticalFault := 
    bPressureHigh OR bTempHigh OR bLevelHigh OR bLevelLow OR
    bAgitatorFailure OR bFeedFlowLoss OR bCoolingWaterLoss OR
    bPHDeviation OR bGasLeakDetected OR bEmergencyStop OR
    bFlameFailure OR bPowerFailure OR bOverfill OR bSensorFault;

// Trigger general emergency response if any fault or E-stop
IF bAnyCriticalFault OR bEmergencyStop THEN
    bESDTriggered := TRUE;
    bLocalAlarm_Sound := TRUE;
    bSCADA_Alarm_Send := TRUE;
    bLockoutAllInputs := TRUE;
    bLogEvent := TRUE;
END_IF;

// C01 - High Pressure
IF bPressureHigh THEN
    vReliefValve_Open := TRUE;
    bEmergencyCooling_ON := TRUE;
    bControlledDepressurization := TRUE;
    bIsolateReactor := TRUE;
END_IF;

// C02 - High Temperature
IF bTempHigh THEN
    bEmergencyCooling_ON := TRUE;
    bAgitator_Stop := TRUE;
    bControlledDepressurization := TRUE;
    bIsolateReactor := TRUE;
END_IF;

// C03 - High Level
IF bLevelHigh THEN
    vFeedValve_Close := TRUE;
    vReliefValve_Open := TRUE;
    bIsolateReactor := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C04 - Low Level
IF bLevelLow THEN
    vFeedValve_Close := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C05 - Agitator Failure
IF bAgitatorFailure THEN
    bAgitator_Stop := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C06 - Feed Flow Loss
IF bFeedFlowLoss THEN
    bFeedPump_Stop := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C07 - Cooling Water Loss
IF bCoolingWaterLoss THEN
    bEmergencyCooling_ON := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C08 - pH Deviation
IF bPHDeviation THEN
    bEmergencyCooling_ON := TRUE;
    vFeedValve_Close := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C09 - Gas Leak Detected
IF bGasLeakDetected THEN
    bVentGasSafely := TRUE;
    bFireSuppression_ON := TRUE;
    bIsolateReactor := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C10 - Emergency Stop (Manual)
IF bEmergencyStop THEN
    vFeedValve_Close := TRUE;
    vReliefValve_Open := TRUE;
    bEmergencyCooling_ON := TRUE;
    bAgitator_Stop := TRUE;
    bFeedPump_Stop := TRUE;
    bESDTriggered := TRUE;
    bLocalAlarm_Sound := TRUE;
    bSCADA_Alarm_Send := TRUE;
    bLockoutAllInputs := TRUE;
    bVentGasSafely := TRUE;
    bIsolateReactor := TRUE;
    bFireSuppression_ON := TRUE;
    bControlledDepressurization := TRUE;
END_IF;

// C11 - Flame Failure
IF bFlameFailure THEN
    bESDTriggered := TRUE;
END_IF;

// C12 - Power Supply Failure
IF bPowerFailure THEN
    bSwitchToBackupPower := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C13 - Overfill Condition
IF bOverfill THEN
    vFeedValve_Close := TRUE;
    vReliefValve_Open := TRUE;
    bIsolateReactor := TRUE;
    bESDTriggered := TRUE;
END_IF;

// C14 - Sensor Fault
IF bSensorFault THEN
    bESDTriggered := TRUE;
    bLockoutAllInputs := TRUE;
    bLogEvent := TRUE;
END_IF;
