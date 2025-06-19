(* Program: Heating System Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Maintains room temperature between 20°C and 22°C using three sensors *)
(* Ensures energy efficiency with hysteresis and handles sensor faults *)
PROGRAM PRG_HeatingControl
VAR
    (* Inputs *)
    TempSensor1 : REAL;               (* Temperature sensor 1, °C *)
    TempSensor2 : REAL;               (* Temperature sensor 2, °C *)
    TempSensor3 : REAL;               (* Temperature sensor 3, °C *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    HeatingOn : BOOL;                 (* TRUE to activate heater *)
    SensorFault : BOOL;               (* TRUE if any sensor reading is invalid *)
    AlarmActive : BOOL;               (* TRUE if sensor fault or emergency stop *)
    
    (* Internal Variables *)
    AvgTemp : REAL;                   (* Average temperature of three sensors *)
    Sensor1Valid : BOOL;              (* TRUE if Sensor1 within 10–30°C *)
    Sensor2Valid : BOOL;              (* TRUE if Sensor2 within 10–30°C *)
    Sensor3Valid : BOOL;              (* TRUE if Sensor3 within 10–30°C *)
END_VAR

(* Initialize outputs *)
HeatingOn := FALSE;                   (* Heater off by default *)
SensorFault := FALSE;                 (* No initial sensor fault *)
AlarmActive := FALSE;                 (* No initial alarm *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt heating and activate alarm *)
    HeatingOn := FALSE;               (* Turn off heater *)
    SensorFault := FALSE;             (* Clear sensor fault *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Validate sensor readings *)
(* Readings outside 10–30°C indicate potential sensor failure *)
Sensor1Valid := TempSensor1 >= 10.0 AND TempSensor1 <= 30.0;
Sensor2Valid := TempSensor2 >= 10.0 AND TempSensor2 <= 30.0;
Sensor3Valid := TempSensor3 >= 10.0 AND TempSensor3 <= 30.0;

(* Check for numerical stability *)
(* Prevent overflow or invalid values *)
IF ABS(TempSensor1) > 1.0E10 OR ABS(TempSensor2) > 1.0E10 OR ABS(TempSensor3) > 1.0E10 THEN
    SensorFault := TRUE;
    HeatingOn := FALSE;               (* Turn off heater *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Check for sensor faults *)
IF NOT Sensor1Valid OR NOT Sensor2Valid OR NOT Sensor3Valid THEN
    (* Any sensor outside range: Flag fault and stop heating *)
    SensorFault := TRUE;
    HeatingOn := FALSE;               (* Turn off heater *)
    AlarmActive := TRUE;              (* Activate alarm *)
ELSE
    (* All sensors valid: Proceed with control *)
    SensorFault := FALSE;
    AlarmActive := FALSE;             (* Clear alarm *)

    (* Compute average temperature *)
    (* Use REAL for precision and to handle decimal division *)
    AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

    (* Check average temperature for numerical validity *)
    IF ABS(AvgTemp) > 1.0E10 THEN
        SensorFault := TRUE;
        HeatingOn := FALSE;           (* Turn off heater *)
        AlarmActive := TRUE;          (* Activate alarm *)
        RETURN;
    END_IF;

    (* Hysteresis-based control *)
    (* Maintain 20–22°C range, avoid frequent switching *)
    IF AvgTemp < 20.0 AND NOT HeatingOn THEN
        (* Below 20°C: Turn heater ON *)
        HeatingOn := TRUE;
    ELSIF AvgTemp > 22.0 AND HeatingOn THEN
        (* Above 22°C: Turn heater OFF *)
        HeatingOn := FALSE;
    END_IF;
    (* Between 20–22°C: Maintain current state *)
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* Heater remains off if fault or emergency *)
IF SensorFault OR EmergencyStop THEN
    HeatingOn := FALSE;
END_IF;

END_PROGRAM
