PROGRAM HeatingControlSystem
VAR_INPUT
    TempSensor1 : REAL; // Temperature reading from sensor 1
    TempSensor2 : REAL; // Temperature reading from sensor 2
    TempSensor3 : REAL; // Temperature reading from sensor 3
END_VAR

VAR_OUTPUT
    HeatingOn : BOOL := FALSE; // Control output for heating system
END_VAR

VAR
    AvgTemp : REAL;           // Average temperature calculated from the three sensors
    SensorFault : BOOL := FALSE; // Flag indicating sensor fault
    LastHeatingState : BOOL;   // Previous state of HeatingOn
END_VAR

// Calculate average temperature
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Fault detection
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR 
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR 
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE;
ELSE
    SensorFault := FALSE;

    // Hysteresis control logic
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;
END_IF;

// Store the current state of HeatingOn for future reference
LastHeatingState := HeatingOn;

// Inline comments explaining the logic:
// The program reads temperature from three sensors and calculates the average temperature.
// It turns the heating ON when the average temperature is below 20°C and OFF when above 22°C.
// Hysteresis is implemented to avoid frequent toggling between ON/OFF states.
// If any sensor reads below 10°C or above 30°C, it treats it as a fault, turns off heating, and flags the error.



