(* IEC 61131-3 Structured Text: PRESSURE_RELIEF_INTERLOCK Function Block *)
(* Purpose: Implements overpressure protection interlock for a pressure vessel *)

FUNCTION_BLOCK PRESSURE_RELIEF_INTERLOCK
VAR_INPUT
    Enable : BOOL;                  (* TRUE to enable interlock logic *)
    PT_Sensor : REAL;               (* Pressure from PT sensor, bar *)
    Valve_Feedback : BOOL;          (* TRUE if relief valve open, FALSE if closed *)
    High_Pressure_Setpoint : REAL;  (* High-pressure threshold, e.g., 15 bar *)
    Reset_Pressure_Setpoint : REAL; (* Reset threshold, e.g., 12 bar *)
END_VAR
VAR_OUTPUT
    Relief_Valve_Open : BOOL;       (* Command to open relief valve *)
    Shutdown_Latch : BOOL;          (* TRUE when in latched shutdown *)
    Alarm_Overpressure : BOOL;      (* TRUE for PT_Sensor > High_Pressure_Setpoint *)
    Alarm_Sensor_Failure : BOOL;    (* TRUE for sensor failure *)
    Alarm_Valve_Malfunction : BOOL; (* TRUE for valve feedback mismatch *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Event logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Normal, 1=Shutdown, 2=Latched *)
    LastPT_Sensor : REAL;           (* Previous PT_Sensor value *)
    SensorStuckTimer : TON;         (* Timer for stuck sensor detection *)
    SensorStuck : BOOL;             (* TRUE if sensor value stuck *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset to safe state *)
    Relief_Valve_Open := TRUE;      (* Open valve *)
    Shutdown_Latch := TRUE;         (* Maintain shutdown *)
    Alarm_Overpressure := FALSE;
    Alarm_Sensor_Failure := FALSE;
    Alarm_Valve_Malfunction := FALSE;
    State := 1;                     (* Shutdown *)
    SensorStuckTimer.IN := FALSE;
ELSE
    CASE State OF
        0: (* Normal Operation *)
            Relief_Valve_Open := FALSE;
            Shutdown_Latch := FALSE;
            Alarm_Overpressure := FALSE;
            Alarm_Sensor_Failure := FALSE;
            Alarm_Valve_Malfunction := FALSE;
            
            (* Validate inputs *)
            IF High_Pressure_Setpoint <= Reset_Pressure_Setpoint OR 
               High_Pressure_Setpoint > 100.0 OR Reset_Pressure_Setpoint < 0.0 THEN
                Relief_Valve_Open := TRUE;
                Shutdown_Latch := TRUE;
                Alarm_Sensor_Failure := TRUE;
                State := 1; (* Move to Shutdown *)
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 19:49:00'; (* Replace with system clock *)
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Setpoints Detected');
                END_IF;
            ELSIF PT_Sensor < 0.0 OR PT_Sensor > 100.0 THEN
                Relief_Valve_Open := TRUE;
                Shutdown_Latch := TRUE;
                Alarm_Sensor_Failure := TRUE;
                State := 1; (* Move to Shutdown *)
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 19:49:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Sensor Failure – PT_Sensor: ', TO_STRING(PT_Sensor));
                END_IF;
            ELSE
                (* Check for stuck sensor *)
                SensorStuckTimer(IN := PT_Sensor = LastPT_Sensor, PT := T#10s);
                IF SensorStuckTimer.Q THEN
                    SensorStuck := TRUE;
                    Relief_Valve_Open := TRUE;
                    Shutdown_Latch := TRUE;
                    Alarm_Sensor_Failure := TRUE;
                    State := 1; (* Move to Shutdown *)
                    IF LogCount < 50 THEN
                        LogCount := LogCount + 1;
                        Timestamp := '2025-05-17 19:49:00';
                        DiagLog[LogCount] := CONCAT(Timestamp, ' Stuck Sensor Detected – PT_Sensor: ', TO_STRING(PT_Sensor));
                    END_IF;
                ELSE
                    (* Check overpressure condition *)
                    IF PT_Sensor > High_Pressure_Setpoint THEN
                        Relief_Valve_Open := TRUE;
                        Shutdown_Latch := TRUE;
                        Alarm_Overpressure := TRUE;
                        State := 1; (* Move to Shutdown *)
                        IF PT_Sensor <> LastPT_Sensor AND LogCount < 50 THEN
                            LogCount := LogCount + 1;
                            Timestamp := '2025-05-17 19:49:00';
                            DiagLog[LogCount] := CONCAT(Timestamp, ' Overpressure Shutdown – PT_Sensor: ', TO_STRING(PT_Sensor));
                        END_IF;
                    END_IF;
                END_IF;
            END_IF;
        
        1: (* Shutdown *)
            Relief_Valve_Open := TRUE;
            Shutdown_Latch := TRUE;
            
            (* Check valve feedback *)
            IF Relief_Valve_Open AND NOT Valve_Feedback THEN
                Alarm_Valve_Malfunction := TRUE;
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 19:49:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Valve Malfunction – Feedback Mismatch');
                END_IF;
            END_IF;
            
            (* Transition to Latched *)
            State := 2;
        
        2: (* Latched *)
            Relief_Valve_Open := TRUE;
            Shutdown_Latch := TRUE;
            
            (* Maintain valve malfunction alarm if present *)
            IF Relief_Valve_Open AND NOT Valve_Feedback THEN
                Alarm_Valve_Malfunction := TRUE;
            END_IF;
            
            (* Check reset conditions *)
            IF PT_Sensor < Reset_Pressure_Setpoint AND 
               PT_Sensor >= 0.0 AND PT_Sensor <= 100.0 AND
               NOT SensorStuck AND Valve_Feedback THEN
                (* Clear alarms and reset system *)
                Alarm_Overpressure := FALSE;
                Alarm_Sensor_Failure := FALSE;
                Alarm_Valve_Malfunction := FALSE;
                Relief_Valve_Open := FALSE;
                Shutdown_Latch := FALSE;
                State := 0; (* Return to Normal *)
                SensorStuckTimer.IN := FALSE;
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 19:49:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' System Reset – PT_Sensor: ', TO_STRING(PT_Sensor));
                END_IF;
            END_IF;
    END_CASE;
    
    (* Update last sensor value *)
    LastPT_Sensor := PT_Sensor;
    
    (* Log buffer overflow check *)
    IF LogCount >= 50 THEN
        LogBufferFull := TRUE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Overpressure protection interlock for a pressure vessel.
   - Inputs:
     - Enable: TRUE to enable interlock logic.
     - PT_Sensor: Pressure (bar, 0–100).
     - Valve_Feedback: TRUE if relief valve open, FALSE if closed.
     - High_Pressure_Setpoint: High-pressure threshold (e.g., 15 bar).
     - Reset_Pressure_Setpoint: Reset threshold (e.g., 12 bar).
   - Outputs:
     - Relief_Valve_Open: Command to open relief valve.
     - Shutdown_Latch: TRUE when in latched shutdown.
     - Alarm_Overpressure: TRUE for PT_Sensor > High_Pressure_Setpoint.
     - Alarm_Sensor_Failure: TRUE for invalid/stuck sensor.
     - Alarm_Valve_Malfunction: TRUE for valve feedback mismatch.
     - DiagLog: ARRAY[1..50] OF STRING[80], event logs.
     - LogCount: Number of log entries.
   - Interlocks:
     - High Pressure (>High_Pressure_Setpoint): Open relief valve, set Shutdown_Latch, issue Alarm_Overpressure.
     - Sensor Failure (invalid/stuck): Open relief valve, set Shutdown_Latch, issue Alarm_Sensor_Failure.
     - Valve Malfunction (feedback mismatch): Maintain shutdown, issue Alarm_Valve_Malfunction.
   - Reset:
     - Shutdown_Latch cleared when PT_Sensor < Reset_Pressure_Setpoint, valid sensor, and valve feedback confirmed.
   - Safety:
     - Bounded log array (50 entries) prevents overflow.
     - State machine ensures scan-cycle safety (e.g., 10–100 ms cycles).
     - Safe state (Relief_Valve_Open, Shutdown_Latch) when disabled or faulty.
     - Input validation (PT_Sensor 0–100 bar, setpoint checks).
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 19:49:00 Overpressure Shutdown – PT_Sensor: 15.5").
     - DiagLog supports HMI/SCADA export.
   - Usage:
     - Overpressure at 15 bar: Opens relief valve, sets Shutdown_Latch, logs event.
     - Sensor failure: Opens relief valve, sets Shutdown_Latch, logs failure.
     - Reset at <12 bar: Clears Shutdown_Latch if no faults, closes valve.
   - Platform Notes:
     - Assumes STRING[80], REAL inputs; adjust for platform limits.
     - Timestamp is static ("2025-05-17 19:49:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Assumes Profibus DP or similar for sensor/valve communication.
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
