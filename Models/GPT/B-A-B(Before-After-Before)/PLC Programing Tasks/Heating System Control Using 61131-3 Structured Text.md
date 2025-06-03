VAR
    TempSensor1, TempSensor2, TempSensor3 : REAL; // Sensor inputs
    AvgTemp : REAL;              // Averaged temperature
    HeatingOn : BOOL := FALSE;   // Heater state
    SensorFault : BOOL := FALSE; // Fault detection flag
END_VAR

// === Average Temperature Calculation ===
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// === Fault Detection ===
// Any sensor outside safe operating range triggers a fault
IF  (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
    (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
    (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN

    SensorFault := TRUE;
    HeatingOn := FALSE;  // Safety action: turn off heating
ELSE
    SensorFault := FALSE;

    // === Heating Control with Hysteresis ===
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;      // Turn on heater if temp is too low
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;     // Turn off heater if temp is high
    END_IF;
END_IF;
