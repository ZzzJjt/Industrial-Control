PROGRAM PLC_PRG
VAR
    // Sensor Inputs
    ExhaustTemp: REAL := 0.0;        // Exhaust gas temperature (°C)
    RotorSpeed: REAL := 0.0;         // % of nominal speed
    CombustionPressure: REAL := 0.0; // Combustion chamber pressure (bar)
    OilPressure: REAL := 0.0;        // Lubrication oil pressure (bar)
    Vibration: REAL := 0.0;          // Bearing vibration (mm/s)
    FlameDetected: BOOL := TRUE;     // Flame status
    FuelPressure: REAL := 0.0;       // Fuel gas pressure (bar)
    CoolantFlow: REAL := 0.0;        // Cooling water flow (L/min)
    SurgeDetected: BOOL := FALSE;    // Compressor surge flag
    EmergencyStop: BOOL := FALSE;    // Manual emergency stop

    // Actuators / Outputs
    TurbineRunning: BOOL := TRUE;
    FuelValveOpen: BOOL := TRUE;
    ReliefValveOpen: BOOL := FALSE;
    BypassValveOpen: BOOL := FALSE;
    AlarmFlameFailure: BOOL := FALSE;
END_VAR

// Interlock Logic
// ----------------

// 1. Overtemperature Interlock
IF ExhaustTemp > 650.0 THEN
    TurbineRunning := FALSE;
END_IF;

// 2. Overspeed Interlock
IF RotorSpeed > 105.0 THEN
    TurbineRunning := FALSE;
END_IF;

// 3. Overpressure Interlock
IF CombustionPressure > 30.0 THEN
    ReliefValveOpen := TRUE;
END_IF;

// 4. Low Lubrication Pressure Interlock
IF OilPressure < 1.5 THEN
    TurbineRunning := FALSE;
END_IF;

// 5. High Vibration Interlock
IF Vibration > 10.0 THEN
    TurbineRunning := FALSE;
END_IF;

// 6. Flame Failure Interlock
IF NOT FlameDetected THEN
    FuelValveOpen := FALSE;
    AlarmFlameFailure := TRUE;
    TurbineRunning := FALSE;
END_IF;

// 7. Low Fuel Gas Pressure Interlock
IF FuelPressure < 2.0 THEN
    FuelValveOpen := FALSE;
    TurbineRunning := FALSE;
END_IF;

// 8. Low Cooling Water Flow Interlock
IF CoolantFlow < 200.0 THEN
    TurbineRunning := FALSE;
END_IF;

// 9. Compressor Surge Interlock
IF SurgeDetected THEN
    BypassValveOpen := TRUE;
    TurbineRunning := FALSE; // Optionally reduce load instead
END_IF;

// 10. Emergency Stop Interlock
IF EmergencyStop THEN
    FuelValveOpen := FALSE;
    TurbineRunning := FALSE;
END_IF;
