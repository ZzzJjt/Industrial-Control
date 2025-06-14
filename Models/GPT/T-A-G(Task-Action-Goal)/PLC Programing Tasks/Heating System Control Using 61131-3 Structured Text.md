VAR
    TempSensor1, TempSensor2, TempSensor3 : REAL; // Input temperature sensors (°C)
    AvgTemp : REAL; // Average temperature
    HeatingOn : BOOL := FALSE; // Output to heater
    SensorFault : BOOL := FALSE; // Fault detection flag
END_VAR

// Compute average of three sensor readings
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Sensor fault detection
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE; // Turn off heating for safety
ELSE
    SensorFault := FALSE;

    // Hysteresis-based control logic
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE; // Turn heating ON if below lower threshold
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE; // Turn heating OFF if above upper threshold
    END_IF;
END_IF;
