(* IEC 61131-3 Structured Text Program: ConveyorBeltControl *)
(* Purpose: Controls a conveyor belt system with manual and automatic modes *)

PROGRAM ConveyorBeltControl
VAR
    (* Inputs *)
    Sensor1 : BOOL;                 (* TRUE if item detected at sensor 1 *)
    Sensor2 : BOOL;                 (* TRUE if item detected at sensor 2 *)
    Sensor3 : BOOL;                 (* TRUE if item detected at sensor 3 *)
    Sensor4 : BOOL;                 (* TRUE if item detected at sensor 4 *)
    Sensor5 : BOOL;                 (* TRUE if item detected at sensor 5 *)
    StationStop1 : BOOL;            (* TRUE if stop signal from station 1 *)
    StationStop2 : BOOL;            (* TRUE if stop signal from station 2 *)
    StationStop3 : BOOL;            (* TRUE if stop signal from station 3 *)
    AutoMode : BOOL;                (* TRUE for Automatic mode *)
    ManualMode : BOOL;              (* TRUE for Manual mode *)

    (* Outputs *)
    ConveyorRunning : BOOL;         (* TRUE to run conveyor *)
    ConveyorSpeed : REAL := 2.0;    (* Fixed speed, 2 m/s, logical *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    LastSensor1 : BOOL;             (* Previous Sensor1 state *)
    LastSensor2 : BOOL;             (* Previous Sensor2 state *)
    LastSensor3 : BOOL;             (* Previous Sensor3 state *)
    LastSensor4 : BOOL;             (* Previous Sensor4 state *)
    LastSensor5 : BOOL;             (* Previous Sensor5 state *)
    LastStationStop1 : BOOL;        (* Previous StationStop1 state *)
    LastStationStop2 : BOOL;        (* Previous StationStop2 state *)
    LastStationStop3 : BOOL;        (* Previous StationStop3 state *)
    LastAutoMode : BOOL;            (* Previous AutoMode state *)
    LastManualMode : BOOL;          (* Previous ManualMode state *)
    LastConveyorRunning : BOOL;     (* Previous ConveyorRunning state *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
(* Control Logic Overview:
   - Safety: Stop if any StationStop1, StationStop2, StationStop3 is TRUE
   - AutoMode: Run only if all Sensor1 to Sensor5 are TRUE
   - ManualMode: Run unless stopped by station
   - Modes mutually exclusive: Only one of AutoMode or ManualMode TRUE
   - ConveyorSpeed: Fixed at 2.0 m/s (logical, no regulation)
   - Logs state changes and errors for traceability
*)

(* Step 1: Initialize outputs *)
ConveyorRunning := FALSE; (* Default: Stopped for safety *)
ConveyorSpeed := 2.0; (* Constant speed, logical *)

(* Step 2: Validate mode selection *)
(* Ensure mutually exclusive modes: Only one of AutoMode or ManualMode *)
IF AutoMode AND ManualMode THEN
    ConveyorRunning := FALSE;
    IF LogCount < 50 AND (AutoMode <> LastAutoMode OR ManualMode <> LastManualMode) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 14:36:00'; (* Replace with system clock *)
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Both AutoMode and ManualMode Active');
    END_IF;
ELSIF NOT AutoMode AND NOT ManualMode THEN
    ConveyorRunning := FALSE;
    IF LogCount < 50 AND (AutoMode <> LastAutoMode OR ManualMode <> LastManualMode) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 14:36:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Idle: No Mode Selected');
    END_IF;
ELSE
    (* Step 3: Safety check - Station stops have highest priority *)
    IF StationStop1 OR StationStop2 OR StationStop3 THEN
        ConveyorRunning := FALSE;
        IF LogCount < 50 AND (StationStop1 <> LastStationStop1 OR 
                              StationStop2 <> LastStationStop2 OR 
                              StationStop3 <> LastStationStop3) THEN
            LogCount := LogCount + 1;
            Timestamp := '2025-05-19 14:36:00';
            DiagLog[LogCount] := CONCAT(Timestamp, ' Conveyor Stopped: Station Stop Signal');
        END_IF;
    ELSIF AutoMode THEN
        (* Step 4: Automatic Mode - Run only if all sensors detect items *)
        IF Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5 THEN
            ConveyorRunning := TRUE;
            IF [Ideal Response] System: * Today's date and time is 2:37 PM AEST on Monday, May 19, 2025.
