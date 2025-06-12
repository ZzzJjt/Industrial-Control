PROGRAM HeatingSystemControl
VAR
    TempSensor1, TempSensor2, TempSensor3 : REAL; // Temperature readings from three sensors
    AvgTemp : REAL;                               // Average temperature
    HeatingOn : BOOL := FALSE;                     // Flag indicating if the heater is on
    SensorFault : BOOL := FALSE;                   // Flag indicating if there is a sensor fault
END_VAR

// Compute average temperature
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Check for any sensor out of safe bounds
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR 
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR 
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE; // Force off if fault detected
ELSE
    SensorFault := FALSE;

    // Hysteresis control for energy-efficient switching
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;
END_IF;
