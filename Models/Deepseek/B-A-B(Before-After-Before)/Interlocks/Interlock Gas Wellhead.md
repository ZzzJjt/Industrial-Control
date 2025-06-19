PROGRAM PLC_PRG
VAR
    // Constants
    MAX_PRESSURE: REAL := 1500.0;      // psi
    MIN_TEMPERATURE: REAL := -50.0;    // °C
    MAX_TEMPERATURE: REAL := 120.0;    // °C
    MIN_FLOW: REAL := 50.0;            // Minimum acceptable flow rate (L/min)

    // Inputs
    PT_101: REAL := 0.0;               // Pressure measurement (psi)
    TT_101: REAL := 0.0;               // Temperature measurement (°C)
    FT_101: REAL := 0.0;               // Flow rate measurement (L/min)
    Manual_Reset: BOOL := FALSE;       // Manual reset after shutdown

    // Outputs
    MV_101_Open: BOOL := TRUE;         // Master Valve (TRUE = Open, FALSE = Closed)
    SHUTDOWN_ACTIVE: BOOL := FALSE;    // Indicates active shutdown state

    // Internal flags
    Pressure_High: BOOL := FALSE;
    Flow_Low: BOOL := FALSE;
    Temp_High: BOOL := FALSE;
END_VAR

// Evaluate trigger conditions
Pressure_High := PT_101 > MAX_PRESSURE;
Flow_Low := FT_101 < MIN_FLOW;
Temp_High := TT_101 > MAX_TEMPERATURE;

// Main interlock logic
IF NOT Manual_Reset THEN
    IF Pressure_High OR Flow_Low THEN
        MV_101_Open := FALSE;
        SHUTDOWN_ACTIVE := TRUE;
    END_IF;

    IF Temp_High THEN
        SHUTDOWN_ACTIVE := TRUE;
        MV_101_Open := FALSE;
    END_IF;
ELSE
    // Manual reset clears the shutdown condition
    SHUTDOWN_ACTIVE := FALSE;
END_IF;

// Prevent automatic restart until manual reset is confirmed
IF SHUTDOWN_ACTIVE THEN
    MV_101_Open := FALSE;
END_IF;
