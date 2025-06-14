(* Prilling Station Interlocks for Ammonium Nitrate *)

(* 1. Overtemperature in Prilling Tower *)
IF Tower_Temp > 180.0 THEN
SHUTDOWN_FEED := TRUE;
ACTIVATE_EMERGENCY_VENT := TRUE;
ALARM_OVERHEAT := TRUE;
END_IF;

(* 2. Underpressure in Ventilation System *)
IF Ventilation_Pressure < 0.5 THEN
STOP_PRILLING_FAN := TRUE;
SHUTDOWN_FEED := TRUE;
ALARM_VENT_FAIL := TRUE;
END_IF;

(* 3. Overpressure in Melt Supply Line *)
IF Melt_Line_Pressure > 10.0 THEN
OPEN_PRESSURE_RELIEF_VALVE := TRUE;
SHUTDOWN_FEED := TRUE;
ALARM_OVERPRESSURE := TRUE;
END_IF;

(* 4. Loss of Airflow in Fluidized Bed *)
IF Airflow < 50.0 THEN
STOP_FLUIDIZATION := TRUE;
SHUTDOWN_FEED := TRUE;
ALARM_NO_AIRFLOW := TRUE;
END_IF;

(* 5. Ammonium Nitrate Melt Temperature Too High *)
IF Melt_Temp > 185.0 THEN
SHUTDOWN_FEED := TRUE;
ALARM_MELT_TEMP := TRUE;
END_IF;

(* 6. Pellet Collector Blockage Detected *)
IF Collector_PressureDrop > 2.5 THEN
STOP_COLLECTOR_FAN := TRUE;
ALARM_BLOCKAGE := TRUE;
END_IF;

(* 7. Feed Pump Failure *)
IF Feed_Pump_Status = FALSE THEN
SHUTDOWN_FEED := TRUE;
ALARM_FEED_FAILURE := TRUE;
END_IF;

(* 8. Emergency Stop *)
IF E_STOP = TRUE THEN
SHUTDOWN_FEED := TRUE;
STOP_ALL_MACHINES := TRUE;
OPEN_ALL_RELIEF_VALVES := TRUE;
ALARM_ESTOP_TRIGGERED := TRUE;
END_IF;

(* 9. High Level in Melt Tank *)
IF Melt_Tank_Level > 90.0 THEN
CLOSE_FEED_VALVE := TRUE;
ALARM_LEVEL_HIGH := TRUE;
END_IF;

(* 10. Prilling Drum Rotation Fault *)
IF Drum_Speed < MIN_DRUM_SPEED THEN
SHUTDOWN_FEED := TRUE;
ALARM_DRUM_FAULT := TRUE;
END_IF;
