FUNCTION_BLOCK FB_ReactorInterlock
VAR_INPUT
    // Cause Conditions (Inputs)
    bPressureHigh: BOOL;           // C01
    bTempHigh: BOOL;               // C02
    bLevelHigh: BOOL;              // C03
    bLevelLow: BOOL;               // C04
    bAgitatorFailure: BOOL;        // C05
    bFeedFlowLoss: BOOL;           // C06
    bCoolingWaterLoss: BOOL;       // C07
    bSensorFault: BOOL;            // C08
    bHighReacConcentration: BOOL;  // C09
    bPowerFailure: BOOL;           // C10
    bGasLeakDetected: BOOL;        // C11
    bEmergencyStop: BOOL;          // C12
    bFlameFailure: BOOL;           // C13
    bOverfill: BOOL;               // C14
    bPHDeviation: BOOL;            // C15
END_VAR

VAR_OUTPUT
    // Process Actions (Outputs)
    vFeedValve_Close: BOOL;                // A01
    vReliefValve_Open: BOOL;               // A02
    bEmergencyCooling_ON: BOOL;            // A03
    bAgitator_Stop: BOOL;                  // A04
    bEmergencyPump_ON: BOOL;               // A05
    bESDTriggered: BOOL;                    // A06
    bLocalAlarm_Sound: BOOL;               // A07
    bSCADA_Alarm_Send: BOOL;               // A08
    bLockoutAllInputs: BOOL;               // A09
    bVentGasSafely: BOOL;                   // A10
    bIsolateReactor: BOOL;                  // A11
    bSwitchToBackupPower: BOOL;            // A12
    bLogEvent: BOOL;                        // A13
    bFireSuppression_ON: BOOL;             // A14
    bControlledDepressurization: BOOL;     // A15
END_VAR

VAR
    // Internal flags
    bAnyCriticalFault: BOOL;
END_VAR
