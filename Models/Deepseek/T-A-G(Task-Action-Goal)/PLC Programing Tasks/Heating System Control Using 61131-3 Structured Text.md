PROGRAM HeatingSystemControl
VAR_INPUT
    TempSensor1 : REAL; // Temperature sensor 1 reading in °C
    TempSensor2 : REAL; // Temperature sensor 2 reading in °C
    TempSensor3 : REAL; // Temperature sensor 3 reading in °C
END_VAR

VAR_OUTPUT
    HeatingOn : BOOL := FALSE; // Output to control the heater
END_VAR

VAR
    AvgTemp : REAL; // Average temperature calculated from the sensors
    SensorFault : BOOL := FALSE; // Flag to indicate sensor fault
END_VAR

// Calculate the average temperature
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Sensor fault detection
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR 
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR 
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE; // Turn off heating if any sensor is faulty
ELSE
    SensorFault := FALSE;

    // Hysteresis-based control
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;
END_IF;

// Additional comments for clarity
// - Calculate the average temperature from the three sensors.
// - Detect sensor faults by checking if any sensor reads below 10°C or above 30°C.
// - If a fault is detected, turn off the heating and set the SensorFault flag.
// - Use hysteresis to control the heating:
//   - Turn heating ON if the average temperature is below 20°C and no fault is present.
//   - Turn heating OFF if the average temperature is above 22°C and no fault is present.
//   - Maintain the current state when the average temperature is between 20°C and 22°C to avoid frequent switching.



