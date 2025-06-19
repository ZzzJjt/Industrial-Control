PROGRAM TemperatureControl
VAR
    // Inputs from three temperature sensors
    TempSensor1 : REAL;
    TempSensor2 : REAL;
    TempSensor3 : REAL;

    // Outputs
    HeatingOn   : BOOL := FALSE;

    // Internal variables
    AvgTemp     : REAL;
    SensorFault : BOOL := FALSE;
END_VAR

// === Step 1: Fault Detection ===
// If any sensor reads below 10°C or above 30°C → treat as fault
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN

    SensorFault := TRUE;
    HeatingOn := FALSE; // Turn off heating for safety

ELSE
    SensorFault := FALSE;

    // === Step 2: Average Temperature Calculation ===
    AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

    // === Step 3: Hysteresis Control Logic ===
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;
END_IF;
