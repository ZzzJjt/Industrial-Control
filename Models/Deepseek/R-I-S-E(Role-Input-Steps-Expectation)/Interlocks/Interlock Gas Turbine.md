FUNCTION_BLOCK FB_GasTurbineInterlock
VAR_INPUT
    // Sensor Inputs
    rExhaustTemp: REAL;          // TT-EXH1 – Exhaust Temp (°C)
    rSpeedRPM: REAL;             // ST-SPD1 – Rotor Speed (RPM)
    rLubeOilPressure: REAL;      // PT-LOP1 – Lube Oil Pressure (bar)
    rVibration: REAL;            // VT-VIB1 – Vibration (mm/s RMS)
    bFlameDetected: BOOL;        // FG-FLM1 – Flame present?
    rFuelGasPressure: REAL;      // PT-FGP1 – Fuel Gas Pressure (bar)
    rCoolingWaterFlow: REAL;     // FT-CWF1 – Cooling Water Flow (%)
    rCompressorDP: REAL;         // DP-CMP1 – Compressor Differential Pressure
    rCombustionChamberPress: REAL; // PT-CMB1 – Combustion Chamber Pressure (bar)
    bEmergencyStop: BOOL;        // ESD_PB1 – Manual Emergency Stop
END_VAR

VAR_OUTPUT
    // Actuator/Control Outputs
    bTripTurbine: BOOL;              // Emergency shutdown signal
    bFuelValve_Close: BOOL;          // Close fuel valve
    bOpenReliefValve: BOOL;          // Open combustion chamber PRV
    bActivateAntiSurgeValve: BOOL;   // Open anti-surge valve
    bStartCooldownSequence: BOOL;    // Begin controlled cooldown
    bAlarmTriggered: BOOL;           // General alarm output
    bLocal_Alarm: BOOL;              // Local alarm activation
    bSCADA_Alarm: BOOL;              // SCADA notification
    bIsolateFuel: BOOL;              // Cut off all fuel supply
    bInitiatePurge: BOOL;            // Start purge sequence after flame failure
END_VAR

VAR
    bAnyCriticalFault: BOOL;
END_VAR

// Initialize outputs at start of scan
bTripTurbine := FALSE;
bFuelValve_Close := FALSE;
bOpenReliefValve := FALSE;
bActivateAntiSurgeValve := FALSE;
bStartCooldownSequence := FALSE;
bAlarmTriggered := FALSE;
bLocal_Alarm := FALSE;
bSCADA_Alarm := FALSE;
bIsolateFuel := FALSE;
bInitiatePurge := FALSE;

// Detect if any critical condition exists
bAnyCriticalFault :=
    rExhaustTemp > 650 OR
    rSpeedRPM > 105.0 OR
    rLubeOilPressure < 1.5 OR
    rVibration > 7.5 OR
    NOT bFlameDetected OR
    rFuelGasPressure < 2.0 OR
    rCoolingWaterFlow < 80.0 OR
    bEmergencyStop;

// General emergency response if any fault or E-stop
IF bAnyCriticalFault THEN
    bAlarmTriggered := TRUE;
    bLocal_Alarm := TRUE;
    bSCADA_Alarm := TRUE;
    bIsolateFuel := TRUE;
    bTripTurbine := TRUE;
END_IF;

// IL01 – Overtemperature Protection
IF rExhaustTemp > 650 THEN
    bStartCooldownSequence := TRUE;
    bTripTurbine := TRUE;
END_IF;

// IL02 – Overspeed Protection
IF rSpeedRPM > 105.0 THEN
    bTripTurbine := TRUE;
END_IF;

// IL03 – Low Lube Oil Pressure
IF rLubeOilPressure < 1.5 THEN
    bTripTurbine := TRUE;
END_IF;

// IL04 – High Vibration
IF rVibration > 7.5 THEN
    bTripTurbine := TRUE;
END_IF;

// IL05 – Flame Failure
IF NOT bFlameDetected THEN
    bIsolateFuel := TRUE;
    bInitiatePurge := TRUE;
    bTripTurbine := TRUE;
END_IF;

// IL06 – Low Fuel Gas Pressure
IF rFuelGasPressure < 2.0 THEN
    bFuelValve_Close := TRUE;
    bTripTurbine := TRUE;
END_IF;

// IL07 – Low Cooling Water Flow
IF rCoolingWaterFlow < 80.0 THEN
    bStartCooldownSequence := TRUE;
    bTripTurbine := TRUE;
END_IF;

// IL08 – Compressor Surge Detection
IF rCompressorDP > 15.0 AND rCombustionChamberPress < 20.0 THEN
    bActivateAntiSurgeValve := TRUE;
END_IF;

// IL09 – Overpressure in Combustion Chamber
IF rCombustionChamberPress > 30.0 THEN
    bOpenReliefValve := TRUE;
    bTripTurbine := TRUE;
END_IF;

// IL10 – Emergency Stop
IF bEmergencyStop THEN
    bTripTurbine := TRUE;
    bIsolateFuel := TRUE;
    bFuelValve_Close := TRUE;
    bOpenReliefValve := TRUE;
    bLocal_Alarm := TRUE;
    bSCADA_Alarm := TRUE;
    bInitiatePurge := TRUE;
END_IF;
