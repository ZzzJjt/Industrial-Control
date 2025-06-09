FUNCTION_BLOCK HeatingControl
VAR_INPUT
    TempSensor1 : REAL; // Temperature sensor 1
    TempSensor2 : REAL; // Temperature sensor 2
    TempSensor3 : REAL; // Temperature sensor 3
END_VAR

VAR_OUTPUT
    HeatingOn    : BOOL := FALSE; // Controls heating system
    SensorFault  : BOOL := FALSE; // Sensor fault flag
END_VAR

VAR
    AvgTemp : REAL := 0.0; // Average temperature
END_VAR

// === Calculate average temperature ===
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// === Fault detection ===
// Any sensor outside 10–30°C triggers fault and disables heating
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN

    SensorFault := TRUE;
    HeatingOn := FALSE;

ELSE
    SensorFault := FALSE;

    // === Hysteresis-based temperature control ===
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE; // Turn heating ON below 20°C
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE; // Turn heating OFF above 22°C
    END_IF;

END_IF;
