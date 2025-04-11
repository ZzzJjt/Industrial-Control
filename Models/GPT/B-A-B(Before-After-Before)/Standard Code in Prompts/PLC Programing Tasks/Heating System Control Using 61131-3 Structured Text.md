```
PROGRAM HeatingSystemControl
VAR
    // Input sensors
    TempSensor1 : REAL; // Temperature sensor 1 reading
    TempSensor2 : REAL; // Temperature sensor 2 reading
    TempSensor3 : REAL; // Temperature sensor 3 reading
    
    // Intermediate variables
    AvgTemp : REAL; // Average temperature calculated from the sensors
    HeatingOn : BOOL := FALSE; // Indicates whether the heater is ON
    HeaterFault : BOOL := FALSE; // Indicates if there is a fault condition
    MinTemp : REAL := 20.0; // Minimum desired temperature
    MaxTemp : REAL := 22.0; // Maximum desired temperature
    Hysteresis : REAL := 0.5; // Temperature hysteresis to minimize switching
    HeaterCycleTime : TIME := T#5s; // Time delay to prevent rapid switching
    HeaterCycleTimer : TIME := T#0s; // Timer for heater cycling
END_VAR

// Calculate average temperature from the three sensors
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Check for sensor faults
IF (TempSensor1 < 0.0 OR TempSensor1 > 50.0) OR (TempSensor2 < 0.0 OR TempSensor2 > 50.0) OR (TempSensor3 < 0.0 OR TempSensor3 > 50.0) THEN
    HeaterFault := TRUE;
ELSIF AvgTemp < MinTemp - Hysteresis THEN
    // Turn the heater ON if below minimum temperature minus hysteresis
    HeatingOn := TRUE;
ELSIF AvgTemp > MaxTemp + Hysteresis THEN
    // Turn the heater ON if above maximum temperature plus hysteresis
    HeatingOn := TRUE;
ELSIF AvgTemp < MinTemp THEN
    // Turn the heater ON if below minimum temperature but within hysteresis
    HeatingOn := TRUE;
ELSIF AvgTemp > MaxTemp THEN
    // Turn the heater OFF if above maximum temperature but within hysteresis
    HeatingOn := FALSE;
ELSIF HeatingOn AND AvgTemp <= MaxTemp - Hysteresis THEN
    // Turn the heater OFF if the temperature drops below max temp minus hysteresis
    HeatingOn := FALSE;
ELSIF NOT HeatingOn AND AvgTemp >= MinTemp + Hysteresis THEN
    // Turn the heater ON if the temperature rises above min temp plus hysteresis
    HeatingOn := TRUE;
ELSIF HeatingOn THEN
    // If the heater is ON, wait for the cycle time before checking again
    HeaterCycleTimer := HeaterCycleTimer + T#1s;
    IF HeaterCycleTimer >= HeaterCycleTime THEN
        HeatingOn := FALSE;
        HeaterCycleTimer := T#0s;
    END_IF;
ELSIF NOT HeatingOn THEN
    // If the heater is OFF, wait for the cycle time before checking again
    HeaterCycleTimer := HeaterCycleTimer + T#1s;
    IF HeaterCycleTimer >= HeaterCycleTime THEN
        HeatingOn := TRUE;
        HeaterCycleTimer := T#0s;
    END_IF;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("Average Temperature", AvgTemp);
// Example: Write("Heating On", HeatingOn);
// Example: Write("Heater Fault", HeaterFault);
// Example: Write("Heater Cycle Timer", HeaterCycleTimer);

END_PROGRAM
```
