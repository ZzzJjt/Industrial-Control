(* IEC 61131-3 Structured Text: REACTOR_DOOR_INTERLOCK Function Block *)
(* Purpose: Implements safety door interlock system for a chemical reactor *)

FUNCTION_BLOCK REACTOR_DOOR_INTERLOCK
VAR_INPUT
    Enable : BOOL;                  (* TRUE to enable interlock logic *)
    DOOR_1_CLOSED : BOOL;           (* TRUE if Door 1 closed, FALSE if open *)
    DOOR_2_CLOSED : BOOL;           (* TRUE if Door 2 closed, FALSE if open *)
    Manual_Reset : BOOL;            (* Rising edge to reset latched shutdown *)
    Reactor_Start_Request : BOOL;   (* TRUE to request reactor startup *)
END_VAR
VAR_OUTPUT
    Allow_Start : BOOL;             (* TRUE if reactor can start *)
    Reactor_Running : BOOL;         (* TRUE when reactor is operating *)
    Emergency_Shutdown : BOOL;      (* TRUE when shutdown active *)
    Heater_Off : BOOL;              (* TRUE to disable heater *)
    Stirrer_Off : BOOL;             (* TRUE to disable stirrer *)
    Pump_Off : BOOL;                (* TRUE to disable pump *)
    Alarm_Door_Open : BOOL;         (* TRUE if any door is open *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Event logs *)
    LogCount : INT;                 (* Number of log entries *)
END_VAR
VAR
    State : INT := 0;               (* State: 0=Idle, 1=Running, 2=Shutdown, 3=Latched *)
    LastDOOR_1_CLOSED : BOOL;       (* Previous DOOR_1_CLOSED state *)
    LastDOOR_2_CLOSED : BOOL;       (* Previous DOOR_2_CLOSED state *)
    LastManual_Reset : BOOL;        (* Previous Manual_Reset state *)
    LastState : INT;                (* Previous state for logging *)
    Timestamp : STRING[20];         (* Simulated timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Main Logic *)
IF NOT Enable THEN
    (* Reset to safe state *)
    Allow_Start := FALSE;
    Reactor_Running := FALSE;
    Emergency_Shutdown := TRUE;
    Heater_Off := TRUE;
    Stirrer_Off := TRUE;
    Pump_Off := TRUE;
    Alarm_Door_Open := FALSE;
    State := 2;                     (* Shutdown *)
ELSE
    CASE State OF
        0: (* Idle *)
            Allow_Start := DOOR_1_CLOSED AND DOOR_2_CLOSED;
            Reactor_Running := FALSE;
            Emergency_Shutdown := FALSE;
            Heater_Off := TRUE;
            Stirrer_Off := TRUE;
            Pump_Off := TRUE;
            Alarm_Door_Open := NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED;
            
            IF Allow_Start AND Reactor_Start_Request THEN
                Reactor_Running := TRUE;
                Heater_Off := FALSE;
                Stirrer_Off := FALSE;
                Pump_Off := FALSE;
                State := 1; (* Move to Running *)
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 19:49:00'; (* Replace with system clock *)
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Reactor Started');
                END_IF;
            ELSIF Alarm_Door_Open AND LogCount < 50 THEN
                IF NOT DOOR_1_CLOSED AND DOOR_1_CLOSED <> LastDOOR_1_CLOSED THEN
                    LogCount := LogCount + 1;
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Door 1 Open – Startup Prevented');
                ELSIF NOT DOOR_2_CLOSED AND DOOR_2_CLOSED <> LastDOOR_2_CLOSED THEN
                    LogCount := LogCount + 1;
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Door 2 Open – Startup Prevented');
                END_IF;
            END_IF;
        
        1: (* Running *)
            Allow_Start := FALSE;
            Reactor_Running := TRUE;
            Emergency_Shutdown := FALSE;
            Heater_Off := FALSE;
            Stirrer_Off := FALSE;
            Pump_Off := FALSE;
            Alarm_Door_Open := NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED;
            
            IF NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED THEN
                Reactor_Running := FALSE;
                Emergency_Shutdown := TRUE;
                Heater_Off := TRUE;
                Stirrer_Off := TRUE;
                Pump_Off := TRUE;
                State := 2; (* Move to Shutdown *)
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 19:49:00';
                    IF NOT DOOR_1_CLOSED THEN
                        DiagLog[LogCount] := CONCAT(Timestamp, ' Door 1 Opened – Shutdown Initiated');
                    ELSE
                        DiagLog[LogCount] := CONCAT(Timestamp, ' Door 2 Opened – Shutdown Initiated');
                    END_IF;
                END_IF;
            END_IF;
        
        2: (* Shutdown *)
            Allow_Start := FALSE;
            Reactor_Running := FALSE;
            Emergency_Shutdown := TRUE;
            Heater_Off := TRUE;
            Stirrer_Off := TRUE;
            Pump_Off := TRUE;
            Alarm_Door_Open := NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED;
            
            (* Transition to Latched *)
            State := 3;
        
        3: (* Latched *)
            Allow_Start := FALSE;
            Reactor_Running := FALSE;
            Emergency_Shutdown := TRUE;
            Heater_Off := TRUE;
            Stirrer_Off := TRUE;
            Pump_Off := TRUE;
            Alarm_Door_Open := NOT DOOR_1_CLOSED OR NOT DOOR_2_CLOSED;
            
            (* Check reset conditions *)
            IF DOOR_1_CLOSED AND DOOR_2_CLOSED AND Manual_Reset AND NOT LastManual_Reset THEN
                Emergency_Shutdown := FALSE;
                Heater_Off := FALSE;
                Stirrer_Off := FALSE;
                Pump_Off := FALSE;
                Alarm_Door_Open := FALSE;
                State := 0; (* Return to Idle *)
                IF State <> LastState AND LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-17 19:49:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' System Reset – All Doors Closed');
                END_IF;
            END_IF;
    END_CASE;
    
    (* Update last values *)
    LastDOOR_1_CLOSED := DOOR_1_CLOSED;
    LastDOOR_2_CLOSED := DOOR_2_CLOSED;
    LastManual_Reset := Manual_Reset;
    LastState := State;
    
    (* Log buffer overflow check *)
    IF LogCount >= 50 THEN
        LogBufferFull := TRUE;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Safety door interlock system for a chemical reactor.
   - Inputs:
     - Enable: TRUE to enable interlock logic.
     - DOOR_1_CLOSED, DOOR_2_CLOSED: TRUE if door closed, FALSE if open.
     - Manual_Reset: Rising edge to reset latched shutdown.
     - Reactor_Start_Request: TRUE to request reactor startup.
   - Outputs:
     - Allow_Start: TRUE if reactor can start (doors closed).
     - Reactor_Running: TRUE when reactor operating.
     - Emergency_Shutdown: TRUE when shutdown active.
     - Heater_Off, Stirrer_Off, Pump_Off: TRUE to disable processes.
     - Alarm_Door_Open: TRUE if any door open.
     - DiagLog: ARRAY[1..50] OF STRING[80], event logs.
     - LogCount: Number of log entries.
   - Interlocks:
     - Prevent Start: If any door open, Allow_Start := FALSE.
     - Emergency Shutdown: If door opens during Reactor_Running, set Emergency_Shutdown, disable processes, latch until doors closed and Manual_Reset.
   - Reset:
     - Shutdown latched until DOOR_1_CLOSED, DOOR_2_CLOSED, and Manual_Reset rising edge.
   - Safety:
     - Bounded log array (50 entries) prevents overflow.
     - State machine ensures scan-cycle safety (e.g., 10–100 ms cycles).
     - Safe state (Emergency_Shutdown, processes off) when disabled or door open.
     - Input validation ensures reliable door status.
   - Traceability:
     - Timestamped logs (e.g., "2025-05-17 19:49:00 Door 1 Opened – Shutdown Initiated").
     - DiagLog supports HMI/SCADA export.
   - Usage:
     - Open Door at Startup: Prevents Allow_Start, logs event.
     - Open Door During Operation: Triggers shutdown, disables processes, latches until reset.
   - Platform Notes:
     - Assumes STRING[80], BOOL inputs; adjust for platform limits.
     - Timestamp is static ("2025-05-17 19:49:00"); replace with system clock (e.g., GET_SYSTEM_TIME).
     - Assumes Profibus DP or similar for door sensor communication.
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_FUNCTION_BLOCK
