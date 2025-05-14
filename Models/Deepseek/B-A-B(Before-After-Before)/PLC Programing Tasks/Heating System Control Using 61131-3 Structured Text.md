PROGRAM PLC_PRG
TITLE 'Temperature Control System with Fault Handling'

(*
    Description:
    Controls a heating system based on averaged temperature from 3 sensors.
    
    Features:
    - Calculates average temperature
    - Turns heater ON below 20°C and OFF above 22°C (hysteresis)
    - Detects sensor faults (values outside 10–30°C range)
    - Disables heater if any sensor fails or reads extreme values
    
    Safety:
    - Prevents rapid cycling via hysteresis
    - Ensures safe behavior when sensors fail
*)

VAR
    // Inputs: Temperature Sensors
    TempSensor1 : REAL := 25.0;   // Example default: room temp
    TempSensor2 : REAL := 24.8;
    TempSensor3 : REAL := 25.2;

    // Internal Logic
    AvgTemp : REAL := 25.0;       // Average of the three sensors
    SensorFault : BOOL := FALSE;  // TRUE if any sensor is out of valid range

    // Output: Heater control
    HeatingOn : BOOL := FALSE;    // TRUE = Heater is active
END_VAR

// === MAIN LOGIC ===

// Step 1: Calculate average temperature
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

// Step 2: Check for sensor faults (any value outside 10–30 °C)
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE;  // Turn off heater for safety
ELSE
    SensorFault := FALSE;

    // Step 3: Heating control logic with hysteresis
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;
END_IF;

END_PROGRAM
