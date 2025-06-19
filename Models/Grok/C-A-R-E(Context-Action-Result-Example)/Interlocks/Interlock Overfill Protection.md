(* IEC 61131-3 Structured Text: OVERFILL_PROTECTION Function Block *)
(* Purpose: Implements overfill protection interlock for a vessel *)

FUNCTION_BLOCK OVERFILL_PROTECTION
VAR_INPUT
    Enable : BOOL;                  (* TRUE to enable interlock logic *)
    Level_Sensor : REAL;            (* Liquid level from LT sensor, % *)
    Valve_Feedback : BOOL;          (* TRUE if inlet valve closed, FALSE if open *)
    High_Level_Setpoint : REAL;     (* High-level threshold, e.g., 95% *)
    Reset_Level_Setpoint : REAL;    (* Reset threshold, e.g., 85% *)
END_VAR
VAR_OUTPUT
    Inlet_Valve_Closed : BOOL;      (* Command to close inlet valve *)
    Shutdown_Latch : BOOL;          (* TRUE when in latched shutdown *)
    Alarm_Overfill : BOOL;          (* TRUE for Level_Sensor > High_Level_Setpoint *)
    Alarm_Sensor_Failure : BOOL;    (* TRUE for sensor failure *)
    Alarm_Valve_Malfunction : BOOL; (* TRUE for valve feedback mismatch *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Event logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Normal, 1=Shutdown, 2=Latched *)
    LastLevel_Sensor : REAL;        (* Previous Level_Sensor value *)
    SensorStuckTimer : TON;         (* Timer for stuck sensor detection *)
    SensorStuck : BOOL;             (* TRUE if sensor value stuck *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset to safe state *)
    Inlet_Valve_Closed := TRUE;     (* Close valve *)
    Shutdown_Latch := TRUE;         (* Maintain shutdown *)
    Alarm_Overfill := FALSE;
    Alarm_Sensor_Failure := FALSE;
    Alarm_Valve_Malfunction := FALSE;
    State := 1;                     (* Shutdown *)
    SensorStuckTimer.IN := FALSE;
ELSE
    CASE State OF
        0: (* Normal Operation *)
            Inlet_Valve_Closed := FALSE;
            Shutdown_Latch := FALSE;
            Alarm_Overfill := FALSE;
            Alarm_Sensor_Failure := FALSE;
            Alarm_Valve_Malfunction := FALSE;
            
            (* Validate inputs *)
            IF High_Level_Setpoint <= Reset_Level_Setpoint OR 
               High_Level_Setpoint > 100.0 OR Reset_Level_Setpoint < 0.0 THEN
                Inlet_Valve_Closed := TRUE;
                Shutdown_Latch := TRUE;
                Alarm_Sensor_Failure := TRUE;
                State := 1; (* Move to Shutdown *)
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 18:06:00'; (* Replace with system clock *)
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Setpoints Detected');
                END_IF;
            ELSIF Level_Sensor < 0.0 OR Level_Sensor > 100.0 THEN
                Inlet_Valve_Closed := TRUE;
                Shutdown_Latch := TRUE;
                Alarm_Sensor_Failure := TRUE;
                State := 1; (* Move to Shutdown *)
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 18:06:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Sensor Failure – Level_Sensor: ', TO_STRING(Level_Sensor));
                END_IF;
            ELSE
                (* Check for stuck sensor *)
                SensorStuckTimer(IN := Level_Sensor = LastLevel_Sensor, PT := T#10s);
                IF SensorStuckTimer.Q THEN
                    SensorStuck := TRUE;
                    Inlet_Valve_Closed := TRUE;
                    Shutdown_Latch := TRUE;
                    Alarm_Sensor_Failure := TRUE;
                    State := 1; (* Move to Shutdown *)
                    IF LogCount < 50 THEN
                        LogCount := LogCount + 1;
                        Timestamp := '2025-05-17 18:06:00';
                        DiagLog[LogCount] := CONCAT(Timestamp, ' Stuck Sensor Detected – Level_Sensor: ', TO_STRING(Level_Sensor));
                    END_IF;
                ELSE
                    (* Check overfill condition *)
                    IF Level_Sensor > High_Level_Setpoint THEN
                        Inlet_Valve_Closed := TRUE;
                        Shutdown_Latch := TRUE;
                        Alarm_Overfill := TRUE;
                        State := 1; (* Move to Shutdown *)
                        IF Level_Sensor <> LastLevel_Sensor AND LogCount < 50 THEN
                            LogCount := LogCount + 1;
                            Timestamp := '2025-05-17 18:06:00';
                            DiagLog[LogCount] := CONCAT(Timestamp, ' Overfill Shutdown – Level_Sensor: ', TO_STRING(Level_Sensor));
                        END_IF;
                    END_IF;
                END_IF;
            END_IF;
        
        1: (* Shutdown *)
            Inlet_Valve_Closed := TRUE;
            Shutdown_Latch := TRUE;
            
            (* Check valve feedback *)
            IF Inlet_Valve_Closed AND NOT Valve_Feedback THEN
                Alarm_Valve_Malfunction := TRUE;
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 18:06:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Valve Malfunction – Feedback Mismatch');
                END_IF;
            END_IF;
            
            (* Transition to Latched *)
            State := 2;
        
        2: (* Latched *)
            Inlet_Valve_Closed := TRUE;
            Shutdown_Latch := TRUE;
            
            (* Maintain valve malfunction alarm if present *)
            IF Inlet_Valve_Closed AND NOT Valve_Feedback THEN
                Alarm_Valve_Malfunction := TRUE;
            END_IF;
            
            (* Check reset conditions *)
            IF Level_Sensor < Reset_Level_Setpoint AND 
               Level_Sensor >= 0.0 AND Level_Sensor <= 100.0 AND
               NOT SensorStuck AND Valve_Feedback THEN
                (* Clear alarms and reset system *)
                Alarm_Overfill := FALSE;
                Alarm_Sensor_Failure := FALSE;
                Alarm_Valve_Malfunction := FALSE;
                Inlet_Valve_Closed := FALSE;
                Shutdown_Latch := FALSE;
                State := 0; (* Return to Normal *)
                SensorStuckTimer.IN := FALSE;
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 18:06:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' System Reset – Level_Sensor: ', TO_STRING(Level_Sensor));
                END_IF;
            END_IF;
    END_CASE;
    
    (* Update last sensor value *)
    LastLevel_Sensor := Level_Sensor;
    
    (* Log buffer overflow check *)
    IF LogCount >= 50 THEN
        LogBufferFull := TRUE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Overfill protection interlock for a vessel.
   - Inputs:
     - Enable: TRUE to enable interlock logic.
     - Level_Sensor: Liquid level (%, 0–100).
     - Valve_Feedback: TRUE if inlet valve closed, FALSE if open.
     - High_Level_Setpoint: High-level threshold (e.g., 95%).
     - Reset_Level_Setpoint: Reset threshold (e.g., 85%).
   - Outputs:
     - Inlet_Valve_Closed: Command to close inlet valve.
     - Shutdown_Latch: TRUE when in latched shutdown.
     - Alarm_Overfill: TRUE for Level_Sensor > High_Level_Setpoint.
     - Alarm_Sensor_Failure: TRUE for invalid/stuck sensor.
     - Alarm_Valve_Malfunction: TRUE for valve feedback mismatch.
     - DiagLog: ARRAY[1..50] OF STRING[80], event logs.
     - LogCount: Number of log entries.
   - Interlocks:
     - High Level (>High_Level_Setpoint): Close inlet valve, set Shutdown_Latch, issue Alarm_Overfill.
     - Sensor Failure (invalid/stuck): Close inlet valve, set Shutdown_Latch, issue Alarm_Sensor_Failure.
     - Valve Malfunction (feedback mismatch): Maintain shutdown, issue Alarm_Valve_Malfunction.
   - Reset:
     - Shutdown_Latch cleared when Level_Sensor < Reset_Level_Setpoint, valid sensor, and valve feedback confirmed.
   - Safety:
     - Bounded log array (50 entries) prevents overflow.
     - State machine ensures scan-cycle safety (e.g., 10–100 ms cycles).
     - Safe state (Inlet_Valve_Closed, Shutdown_Latch) when disabled or faulty.
     - Input validation (Level_Sensor 0–100%, setpoint checks).
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 18:06:00 Overfill Shutdown – Level_Sensor: 95.5").
     - DiagLog supports HMI/SCADA export.
   - Usage:
     - Overfill at 95%: Closes inlet valve, sets Shutdown_Latch, logs event.
     - Sensor failure: Closes inlet valve, sets Shutdown_Latch, logs failure.
     - Reset at <85%: Clears Shutdown_Latch if no faults, reopens valve.
   - Platform Notes:
     - Assumes STRING[80], REAL inputs; adjust for platform limits.
     - Timestamp is static ("2025-05-17 18:06:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Assumes Profibus DP or similar for sensor/valve communication.
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
