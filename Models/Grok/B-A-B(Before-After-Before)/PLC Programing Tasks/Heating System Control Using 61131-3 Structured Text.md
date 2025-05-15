(* Heating Control Logic in IEC 61131-3 Structured Text *)
(* Purpose: Regulate temperature using averaged sensor inputs with hysteresis and fault handling *)

PROGRAM HeatingControl
VAR
    (* Inputs *)
    TempSensor1 : REAL;              (* Temperature reading from sensor 1 in °C *)
    TempSensor2 : REAL;              (* Temperature reading from sensor 2 in °C *)
    TempSensor3 : REAL;              (* Temperature reading from sensor 3 in °C *)

    (* Outputs *)
    HeatingOn : BOOL := FALSE;       (* TRUE when heating system is active *)

    (* Internal Variables *)
    AvgTemp : REAL;                  (* Average temperature from three sensors *)
    SensorFault : BOOL;              (* TRUE if any sensor is out of range *)
END_VAR

(* Main Control Logic *)
(* 1. Calculate Average Temperature *)
AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

(* 2. Sensor Fault and Range Check *)
(* Sensors are considered faulty if readings are outside 10°C–30°C *)
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR 
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR 
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE;  (* Disable heating for safety during fault *)
ELSE
    SensorFault := FALSE;
    
    (* 3. Heating Control with Hysteresis *)
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;  (* Turn on heating if temperature below 20°C *)
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;  (* Turn off heating if temperature above 22°C *)
    END_IF;
END_IF;

(* Notes:
   - Temperature Control:
     - Heating activates when AvgTemp < 20°C
     - Heating deactivates when AvgTemp > 22°C
     - Hysteresis (20°C–22°C) prevents rapid on/off cycling
   - Fault Handling:
     - Sensor readings outside 10°C–30°C trigger SensorFault
     - Heating is disabled during fault for safety
   - Physical Integration:
     - TempSensor1, TempSensor2, TempSensor3: Analog inputs (e.g., thermocouples or RTDs)
     - HeatingOn: Relay output to control heating system (e.g., furnace or radiator)
   - Scalability: Add more sensors by extending AvgTemp calculation
   - Maintenance:
     - Add HMI to display AvgTemp, SensorFault, and HeatingOn status
     - Log sensor faults for diagnostics
*)
END_PROGRAM
