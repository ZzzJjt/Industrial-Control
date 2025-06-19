PROGRAM PrillingStationInterlocks
VAR
    // Constants
    SAFE_TEMPERATURE: REAL := 150.0; // Safe temperature threshold (°C)
    HIGH_PRESSURE_LIMIT: REAL := 10.0; // High pressure limit (bar)
    MIN_COOLING_AIR_FLOW: REAL := 500.0; // Minimum cooling air flow (m³/h)
    MAX_AMMONIUM_NITRATE_LEVEL: REAL := 90.0; // Maximum level (%)

    // Inputs
    TemperatureSensor: REAL := 0.0;
    PressureSensor: REAL := 0.0;
    CoolingAirFlowSensor: REAL := 0.0;
    AmmoniumNitrateLevelSensor: REAL := 0.0;
    MeltPumpRunning: BOOL := FALSE;
    ScrubberOperational: BOOL := TRUE;

    // Outputs
    PrillTowerOperation: BOOL := TRUE; // TRUE = Running, FALSE = Stopped
    EmergencyStop: BOOL := FALSE;
END_VAR

// Overtemperature Interlock
IF TemperatureSensor > SAFE_TEMPERATURE THEN
    PrillTowerOperation := FALSE;
    // Log event, trigger alarm, etc.
END_IF;

// High Pressure Interlock
IF PressureSensor > HIGH_PRESSURE_LIMIT THEN
    PrillTowerOperation := FALSE;
    // Vent or isolate equipment, log event, trigger alarm, etc.
END_IF;

// Cooling Air Flow Loss Interlock
IF CoolingAirFlowSensor < MIN_COOLING_AIR_FLOW THEN
    PrillTowerOperation := FALSE;
    // Stop melt feed, log event, trigger alarm, etc.
END_IF;

// Ammonium Nitrate Level High Interlock
IF AmmoniumNitrateLevelSensor > MAX_AMMONIUM_NITRATE_LEVEL THEN
    PrillTowerOperation := FALSE;
    // Halt production, log event, trigger alarm, etc.
END_IF;

// Melt Pump Failure Interlock
IF NOT MeltPumpRunning THEN
    PrillTowerOperation := FALSE;
    // Shut down prilling head, log event, trigger alarm, etc.
END_IF;

// Scrubber Failure Interlock
IF NOT ScrubberOperational THEN
    PrillTowerOperation := FALSE;
    // Stop prill tower, log event, trigger alarm, etc.
END_IF;

// Emergency Stop Interlock
IF EmergencyStop THEN
    PrillTowerOperation := FALSE;
    // All major systems are shut down, log event, trigger alarm, etc.
END_IF;

// Additional logic for other interlocks (e.g., power failure, explosion vent activation, inert gas loss) would follow here...
