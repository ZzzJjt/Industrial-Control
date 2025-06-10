PROGRAM HeatingSystemControl
VAR
    // Temperature sensor inputs
    TempSensor1 : REAL;
    TempSensor2 : REAL;
    TempSensor3 : REAL;

    // Calculated average temperature
    AvgTemp : REAL;

    // Outputs and status flags
    HeatingOn : BOOL := FALSE;
    SensorFault : BOOL := FALSE;
END_VAR

// --- Compute average temperature ---
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// --- Sensor Fault Detection Logic ---
// Mark as fault if any sensor reads below 10°C or above 30°C
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN

    SensorFault := TRUE;
    HeatingOn := FALSE;     // Force heating OFF on sensor error

ELSE
    SensorFault := FALSE;

    // --- Hysteresis-Based Heating Control ---
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;  // Turn ON heater below lower threshold
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE; // Turn OFF heater above upper threshold
    END_IF;
END_IF;
