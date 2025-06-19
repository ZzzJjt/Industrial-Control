(* IEC 61131-3 Structured Text: SUBSEA_WELLHEAD_INTERLOCKS Function Block *)
(* Purpose: Implements emergency interlock system for subsea gas wellhead *)

FUNCTION_BLOCK SUBSEA_WELLHEAD_INTERLOCKS
VAR_INPUT
    Enable : BOOL;                  (* TRUE to enable interlock logic *)
    PT_101 : REAL;                  (* Pressure from PT-101, psi *)
    TT_101 : REAL;                  (* Temperature from TT-101, °C *)
    FT_101 : REAL;                  (* Flow rate from FT-101, L/min *)
    Reset : BOOL;                   (* Manual reset on rising edge *)
END_VAR
VAR_OUTPUT
    MV_101_Closed : BOOL;           (* Command to close MV-101 *)
    Shutdown : BOOL;                (* TRUE when system in shutdown *)
    Alarm_HighPressure : BOOL;      (* TRUE for PT_101 > 1500 psi *)
    Alarm_LowFlow : BOOL;           (* TRUE for FT_101 < 10 L/min *)
    Alarm_HighTemp : BOOL;          (* TRUE for TT_101 > 120°C *)
    SystemLocked : BOOL;            (* TRUE when restart locked out *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Event logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Normal, 1=Shutdown, 2=Locked *)
    LastEnable : BOOL;              (* Previous Enable state *)
    LastReset : BOOL;               (* Previous Reset state *)
    LastPT_101 : REAL;              (* Previous PT-101 value *)
    LastTT_101 : REAL;              (* Previous TT-101 value *)
    LastFT_101 : REAL;              (* Previous FT-101 value *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset outputs to safe state *)
    MV_101_Closed := TRUE;          (* Close valve *)
    Shutdown := TRUE;               (* Maintain shutdown *)
    Alarm_HighPressure := FALSE;
    Alarm_LowFlow := FALSE;
    Alarm_HighTemp := FALSE;
    SystemLocked := TRUE;           (* Lock system *)
    State := 2;                     (* Locked *)
ELSE
    CASE State OF
        0: (* Normal Operation *)
            MV_101_Closed := FALSE;
            Shutdown := FALSE;
            Alarm_HighPressure := FALSE;
            Alarm_LowFlow := FALSE;
            Alarm_HighTemp := FALSE;
            SystemLocked := FALSE;
            
            (* Validate inputs *)
            IF PT_101 < 0.0 OR TT_101 < -50.0 OR FT_101 < 0.0 THEN
                MV_101_Closed := TRUE;
                Shutdown := TRUE;
                SystemLocked := TRUE;
                State := 1; (* Move to Shutdown *)
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 18:06:00'; (* Replace with system clock *)
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Invalid Sensor Data Detected');
                END_IF;
            ELSE
                (* Check interlock conditions *)
                IF PT_101 > 1500.0 THEN
                    MV_101_Closed := TRUE;
                    Shutdown := TRUE;
                    Alarm_HighPressure := TRUE;
                    SystemLocked := TRUE;
                    State := 1; (* Move to Shutdown *)
                    IF PT_101 <> LastPT_101 AND LogCount < 50 THEN
                        LogCount := LogCount + 1;
                        Timestamp := '2025-05-17 18:06:00';
                        DiagLog[LogCount] := CONCAT(Timestamp, ' High Pressure Shutdown – PT_101: ', TO_STRING(PT_101));
                    END_IF;
                ELSIF FT_101 < 10.0 THEN
                    MV_101_Closed := TRUE;
                    Shutdown := TRUE;
                    Alarm_LowFlow := TRUE;
                    SystemLocked := TRUE;
                    State := 1; (* Move to Shutdown *)
                    IF FT_101 <> LastFT_101 AND LogCount < 50 THEN
                        LogCount := LogCount + 1;
                        Timestamp := '2025-05-17 18:06:00';
                        DiagLog[LogCount] := CONCAT(Timestamp, ' Low Flow Shutdown – FT_101: ', TO_STRING(FT_101));
                    END_IF;
                ELSIF TT_101 > 120.0 THEN
                    MV_101_Closed := TRUE;
                    Shutdown := TRUE;
                    Alarm_HighTemp := TRUE;
                    SystemLocked := TRUE;
                    State := 1; (* Move to Shutdown *)
                    IF TT_101 <> LastTT_101 AND LogCount < 50 THEN
                        LogCount := LogCount + 1;
                        Timestamp := '2025-05-17 18:06:00';
                        DiagLog[LogCount] := CONCAT(Timestamp, ' High Temperature Shutdown – TT_101: ', TO_STRING(TT_101));
                    END_IF;
                END_IF;
            END_IF;
        
        1: (* Shutdown *)
            MV_101_Closed := TRUE;
            Shutdown := TRUE;
            SystemLocked := TRUE;
            (* Maintain alarms from trigger condition *)
            IF Reset AND NOT LastReset THEN (* Rising edge on Reset *)
                State := 2; (* Move to Locked, awaiting reset completion *)
            END_IF;
        
        2: (* Locked *)
            MV_101_Closed := TRUE;
            Shutdown := TRUE;
            SystemLocked := TRUE;
            IF Reset AND NOT LastReset THEN (* Confirm reset *)
                (* Clear alarms and reset system *)
                Alarm_HighPressure := FALSE;
                Alarm_LowFlow := FALSE;
                Alarm_HighTemp := FALSE;
                MV_101_Closed := FALSE;
                Shutdown := FALSE;
                SystemLocked := FALSE;
                State := 0; (* Return to Normal *)
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 18:06:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' System Reset by Operator');
                END_IF;
            END_IF;
    END_CASE;
    
    (* Update last values for change detection *)
    LastPT_101 := PT_101;
    LastTT_101 := TT_101;
    LastFT_101 := FT_101;
    LastReset := Reset;
    LastEnable := Enable;
    
    (* Log buffer overflow check *)
    IF LogCount >= 50 THEN
        LogBufferFull := TRUE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Emergency interlock system for subsea gas wellhead.
   - Inputs:
     - Enable: TRUE to enable interlock logic.
     - PT_101: Pressure (psi, e.g., 0–2000).
     - TT_101: Temperature (°C, e.g., -50–200).
     - FT_101: Flow rate (L/min, e.g., 0–1000).
     - Reset: Manual reset on rising edge.
   - Outputs:
     - MV_101_Closed: Command to close MV-101.
     - Shutdown: TRUE when system in shutdown.
     - Alarm_HighPressure: TRUE for PT_101 > 1500 psi.
     - Alarm_LowFlow: TRUE for FT_101 < 10 L/min.
     - Alarm_HighTemp: TRUE for TT_101 > 120°C.
     - SystemLocked: TRUE when restart locked out.
     - DiagLog: ARRAY[1..50] OF STRING[80], event logs.
     - LogCount: Number of log entries.
   - Interlocks:
     - High Pressure (>1500 psi): Close MV-101, set Shutdown, issue Alarm_HighPressure.
     - Low Flow (<10 L/min): Close MV-101, set Shutdown, issue Alarm_LowFlow.
     - High Temperature (>120°C): Close MV-101, set Shutdown, issue Alarm_HighTemp.
   - Manual Reset:
     - SystemLocked prevents restart until Reset rising edge.
     - Reset clears alarms, Shutdown, and SystemLocked.
   - Safety:
     - Bounded log array (50 entries) prevents overflow.
     - State machine ensures scan-cycle safety (e.g., 10–100 ms cycles).
     - Safe state (MV_101_Closed, Shutdown) when disabled.
     - Input validation for sensor data (non-negative, reasonable ranges).
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 18:06:00 High Pressure Shutdown – PT_101: 1550.0").
     - DiagLog supports HMI/SCADA export.
   - Usage:
     - High Pressure: Closes MV-101, sets Shutdown, locks system, logs event.
     - Manual Reset: Operator sets Reset=TRUE to re-enable after inspection.
   - Platform Notes:
     - Assumes STRING[80], REAL inputs; adjust for platform limits.
     - Timestamp is static ("2025-05-17 18:06:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Actions assume Profibus DP or similar for subsea communication.
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
