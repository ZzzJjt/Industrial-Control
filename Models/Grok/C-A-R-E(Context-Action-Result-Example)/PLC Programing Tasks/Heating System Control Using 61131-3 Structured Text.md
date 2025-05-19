(* IEC 61131-3 Structured Text Program: HeatingSystemControl *)
(* Purpose: Controls a heating system with three temperature sensors *)

PROGRAM HeatingSystemControl
VAR
    (* Inputs *)
    TempSensor1 : REAL;             (* Temperature sensor 1 reading, °C *)
    TempSensor2 : REAL;             (* Temperature sensor 2 reading, °C *)
    TempSensor3 : REAL;             (* Temperature sensor 3 reading, °C *)

    (* Outputs *)
    HeatingOn : BOOL := FALSE;      (* TRUE to activate heater *)
    SensorFault : BOOL := FALSE;    (* TRUE if any sensor is faulty *)
    AvgTemp : REAL;                 (* Average temperature, °C *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    LastTempSensor1 : REAL;         (* Previous TempSensor1 value *)
    LastTempSensor2 : REAL;         (* Previous TempSensor2 value *)
    LastTempSensor3 : REAL;         (* Previous TempSensor3 value *)
    LastHeatingOn : BOOL;           (* Previous HeatingOn state *)
    LastSensorFault : BOOL;         (* Previous SensorFault state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Calculate AvgTemp = (TempSensor1 + TempSensor2 + TempSensor3) / 3.0
   - Fault Detection: SensorFault := TRUE if any sensor <10.0°C or >30.0°C
   - Hysteresis:
     - HeatingOn := TRUE if HeatingOn = FALSE AND AvgTemp < 20.0
     - HeatingOn := FALSE if HeatingOn = TRUE AND AvgTemp > 22.0
     - Maintain state if 20.0 ≤ AvgTemp ≤ 22.0
   - Safety: HeatingOn := FALSE if SensorFault = TRUE
   - Logs heater state changes and faults for traceability
*)

(* Step 1: Initialize outputs *)
AvgTemp := 0.0;
SensorFault := FALSE;
(* HeatingOn retains last state due to hysteresis *)

(* Step 2: Validate sensor inputs *)
IF NOT IS_VALID_REAL(TempSensor1) OR NOT IS_VALID_REAL(TempSensor2) OR NOT IS_VALID_REAL(TempSensor3) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE;
    AvgTemp := 0.0;
    IF LogCount < 50 AND (TempSensor1 <> LastTempSensor1 OR 
                          TempSensor2 <> LastTempSensor2 OR 
                          TempSensor3 <> LastTempSensor3) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 16:45:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Sensor Reading');
    END_IF;
    (* Update last states *)
    LastTempSensor1 := TempSensor1;
    LastTempSensor2 := TempSensor2;
    LastTempSensor3 := TempSensor3;
    LastSensorFault := SensorFault;
    LastHeatingOn := HeatingOn;
    RETURN;
END_IF;

(* Step 3: Fault detection *)
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR 
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR 
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    SensorFault := TRUE;
    HeatingOn := FALSE;
    AvgTemp := 0.0;
    IF LogCount < 50 AND (TempSensor1 <> LastTempSensor1 OR 
                          TempSensor2 <> LastTempSensor2 OR 
                          TempSensor3 <> LastTempSensor3 OR 
                          SensorFault <> LastSensorFault) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 16:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Sensor Fault Detected: Temp1=', 
            CONCAT(TO_STRING(TempSensor1), CONCAT(', Temp2=', 
            CONCAT(TO_STRING(TempSensor2), CONCAT(', Temp3=', TO_STRING(TempSensor3))))));
    END_IF;
ELSE
    SensorFault := FALSE;
    (* Step 4: Calculate average temperature *)
    AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;

    (* Step 5: Hysteresis control logic *)
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        HeatingOn := FALSE;
    END_IF;

    (* Step 6: Validate average temperature *)
    IF NOT IS_VALID_REAL(AvgTemp) THEN
        SensorFault := TRUE;
        HeatingOn := FALSE;
        AvgTemp := 0.0;
        IF LogCount < 50 THEN
            LogCount := LogCount + 1;
            Timestamp := '2025-05-19 16:45:00';
            DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Invalid Average Temperature');
        END_IF;
    END_IF;
END_IF;

(* Step 7: Log state changes *)
IF HeatingOn AND NOT LastHeatingOn THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 16:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Heater ON: AvgTemp=', TO_STRING(AvgTemp));
    END_IF;
ELSIF NOT HeatingOn AND LastHeatingOn THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 16:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Heater OFF: AvgTemp=', TO_STRING(AvgTemp));
    END_IF;
END_IF;
IF SensorFault AND NOT LastSensorFault THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 16:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Sensor Fault Active');
    END_IF;
ELSIF NOT SensorFault AND LastSensorFault THEN
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 16:45:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Sensor Fault Cleared');
    END_IF;
END_IF;

(* Step 8: Update last states for logging *)
LastTempSensor1 := TempSensor1;
LastTempSensor2 := TempSensor2;
LastTempSensor3 := TempSensor3;
LastHeatingOn := HeatingOn;
LastSensorFault := SensorFault;

(* Step 9: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Controls heating system to maintain 20°C-22°C using three sensors.
   - Inputs:
     - TempSensor1, TempSensor2, TempSensor3: REAL, temperature in °C.
   - Outputs:
     - HeatingOn: BOOL, TRUE to activate heater.
     - SensorFault: BOOL, TRUE if any sensor <10.0°C or >30.0°C.
     - AvgTemp: REAL, average temperature.
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - AvgTemp = (TempSensor1 + TempSensor2 + TempSensor3) / 3.0.
     - Fault: SensorFault := TRUE if any sensor <10.0°C, >30.0°C, or invalid.
     - Hysteresis: HeatingOn := TRUE if AvgTemp < 20.0, FALSE if >22.0.
     - Safety: HeatingOn := FALSE if SensorFault = TRUE.
   - Optimization:
     - Simple logic (~30 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, no loops except logging.
     - Minimal memory (~4 KB for logs, scalars for sensors).
   - Safety:
     - Validates sensor values (finite, 10.0°C-30.0°C).
     - Disables heater on faults.
     - Logs state changes for traceability.
   - Usage:
     - Building automation: Maintains 20°C-22°C, handles sensor faults.
     - Example: AvgTemp=19.8 → HeatingOn=TRUE; TempSensor1=8.0 → SensorFault=TRUE, HeatingOn=FALSE.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL; adjust for platform limits.
     - Timestamp is static; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
