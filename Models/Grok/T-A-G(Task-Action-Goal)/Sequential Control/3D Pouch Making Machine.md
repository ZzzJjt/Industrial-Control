(* Structured Text program for 3D pouch making machine control *)
(* Conforms to IEC 61131-3 standard *)

PROGRAM PouchMachineControl
VAR
    (* Machine State Variables *)
    MachineState: INT := 0; (* 0: Idle, 1: Starting, 2: Running, 3: Stopping, 4: Stopped *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Heating Stations Control *)
    HeatingStations: ARRAY[1..8] OF BOOL; (* Power state of 8 heating stations *)
    HeatingTemps: ARRAY[1..8] OF REAL; (* Current temperatures *)
    HeatingSetpoint: REAL := 180.0; (* Target temperature in °C *)
    HeatingReady: ARRAY[1..8] OF BOOL; (* Station at temperature *)
    
    (* Cooling Stations Control *)
    CoolingStations: ARRAY[1..8] OF BOOL; (* Power state of 8 cooling stations *)
    CoolingTemps: ARRAY[1..8] OF REAL; (* Current temperatures *)
    CoolingSetpoint: REAL := 20.0; (* Target temperature in °C *)
    CoolingReady: ARRAY[1..8] OF BOOL; (* Station at temperature *)
    
    (* Feeder Units Control *)
    Feeder1_Speed: REAL := 0.0; (* Feeder 1 speed in m/min *)
    Feeder2_Speed: REAL := 0.0; (* Feeder 2 speed in m/min *)
    TensionSensor1: REAL; (* Tension feedback in N *)
    TensionSensor2: REAL; (* Tension feedback in N *)
    TensionSetpoint: REAL := 50.0; (* Target tension in N *)
    TensionTolerance: REAL := 5.0; (* Acceptable tension deviation *)
    
    (* Cutter Control *)
    HorizontalCutter: BOOL := FALSE; (* Horizontal cutter activation *)
    VerticalCutter: BOOL := FALSE; (* Vertical cutter activation *)
    MaterialSpeed: REAL; (* Material movement speed in m/min *)
    CutterSyncPulse: BOOL; (* Synchronization signal *)
    
    (* Timers *)
    StartupTimer: TON; (* Timer for startup sequence *)
    ShutdownTimer: TON; (* Timer for shutdown sequence *)
    StationDelay: TON; (* Delay between station activations *)
    
    (* Configuration Parameters *)
    StartupDelay: TIME := T#2s; (* Delay between station startups *)
    ShutdownDelay: TIME := T#3s; (* Delay for cooling during shutdown *)
    TempTolerance: REAL := 5.0; (* Acceptable temperature deviation *)
    MaxStartupTime: TIME := T#60s; (* Maximum startup duration *)
    MaxShutdownTime: TIME := T#90s; (* Maximum shutdown duration *)
END_VAR

(* Main Control Logic *)
CASE MachineState OF
    0: (* Idle State *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            MachineState := 1; (* Initiate startup *)
            StartupTimer(IN := TRUE, PT := MaxStartupTime);
        END_IF;
    
    1: (* Startup Sequence *)
        (* Step 1: Power on heating stations sequentially *)
        FOR i := 1 TO 8 DO
            IF NOT HeatingStations[i] THEN
                IF StationDelay.Q THEN
                    HeatingStations[i] := TRUE;
                    StationDelay(IN := FALSE);
                ELSE
                    StationDelay(IN := TRUE, PT := StartupDelay);
                END_IF;
                EXIT; (* Process one station at a time *)
            END_IF;
        END_FOR;
        
        (* Check if all heating stations are at setpoint *)
        FOR i := 1 TO 8 DO
            HeatingReady[i] := (ABS(HeatingTemps[i] - HeatingSetpoint) <= TempTolerance);
        END_FOR;
        
        (* Step 2: Power on cooling stations *)
        IF FOR_ALL i := 1 TO 8 DO HeatingReady[i] THEN
            FOR i := 1 TO 8 DO
                IF NOT CoolingStations[i] THEN
                    IF StationDelay.Q THEN
                        CoolingStations[i] := TRUE;
                        StationDelay(IN := FALSE);
                    ELSE
                        StationDelay(IN := TRUE, PT := StartupDelay);
                    END_IF;
                    EXIT;
                END_IF;
            END_FOR;
        END_IF;
        
        (* Check if all cooling stations are at setpoint *)
        FOR i := 1 TO 8 DO
            CoolingReady[i] := (ABS(CoolingTemps[i] - CoolingSetpoint) <= TempTolerance);
        END_FOR;
        
        (* Step 3: Start feeder units with tension control *)
        IF FOR_ALL i := 1 TO 8 DO CoolingReady[i] THEN
            Feeder1_Speed := 10.0; (* Initial speed *)
            Feeder2_Speed := 10.0;
            
            (* PID-like tension control *)
            IF ABS(TensionSensor1 - TensionSetpoint) > TensionTolerance THEN
                Feeder1_Speed := Feeder1_Speed + (TensionSetpoint - TensionSensor1) * 0.1;
            END_IF;
            IF ABS(TensionSensor2 - TensionSetpoint) > TensionTolerance THEN
                Feeder2_Speed := Feeder2_Speed + (TensionSetpoint - TensionSensor2) * 0.1;
            END_IF;
        END_IF;
        
        (* Step 4: Synchronize cutters *)
        IF ABS(TensionSensor1 - TensionSetpoint) <= TensionTolerance AND
           ABS(TensionSensor2 - TensionSetpoint) <= TensionTolerance THEN
            IF MaterialSpeed > 0.0 THEN
                CutterSyncPulse := TRUE; (* Trigger cutter synchronization *)
                HorizontalCutter := CutterSyncPulse AND (MaterialSpeed > 0.0);
                VerticalCutter := CutterSyncPulse AND (MaterialSpeed > 0.0);
            END_IF;
        END_IF;
        
        (* Complete startup *)
        IF FOR_ALL i := 1 TO 8 DO (HeatingReady[i] AND CoolingReady[i]) AND
           ABS(TensionSensor1 - TensionSetpoint) <= TensionTolerance AND
           ABS(TensionSensor2 - TensionSetpoint) <= TensionTolerance AND
           CutterSyncPulse THEN
            MachineState := 2; (* Running state *)
            StartupTimer(IN := FALSE);
        END_IF;
        
        (* Timeout check *)
        IF StartupTimer.Q THEN
            FaultState := TRUE;
            MachineState := 4; (* Stopped due to fault *)
        END_IF;
    
    2: (* Running State *)
        (* Maintain tension control *)
        IF ABS(TensionSensor1 - TensionSetpoint) > TensionTolerance THEN
            Feeder1_Speed := Feeder1_Speed + (TensionSetpoint - TensionSensor1) * 0.1;
        END_IF;
        IF ABS(TensionSensor2 - TensionSetpoint) > TensionTolerance THEN
            Feeder2_Speed := Feeder2_Speed + (TensionSetpoint - TensionSensor2) * 0.1;
        END_IF;
        
        (* Monitor for stop command or faults *)
        IF StopCommand OR EmergencyStop OR FaultState THEN
            MachineState := 3; (* Initiate shutdown *)
            ShutdownTimer(IN := TRUE, PT := MaxShutdownTime);
        END_IF;
    
    3: (* Shutdown Sequence *)
        (* Step 1: Stop cutters *)
        HorizontalCutter := FALSE;
        VerticalCutter := FALSE;
        CutterSyncPulse := FALSE;
        
        (* Step 2: Gradually stop feeders *)
        IF Feeder1_Speed > 0.0 OR Feeder2_Speed > 0.0 THEN
            Feeder1_Speed := MAX(0.0, Feeder1_Speed - 0.5);
            Feeder2_Speed := MAX(0.0, Feeder2_Speed - 0.5);
        END_IF;
        
        (* Step 3: Turn off cooling stations *)
        IF Feeder1_Speed = 0.0 AND Feeder2_Speed = 0.0 THEN
            FOR i := 8 DOWNTO 1 DO
                IF CoolingStations[i] THEN
                    IF StationDelay.Q THEN
                        CoolingStations[i] := FALSE;
                        StationDelay(IN := FALSE);
                    ELSE
                        StationDelay(IN := TRUE, PT := ShutdownDelay);
                    END_IF;
                    EXIT;
                END_IF;
            END_FOR;
        END_IF;
        
        (* Step 4: Turn off heating stations after cooling *)
        IF FOR_ALL i := 1 TO 8 DO NOT CoolingStations[i] THEN
            FOR i := 8 DOWNTO 1 DO
                IF HeatingStations[i] THEN
                    IF StationDelay.Q THEN
                        HeatingStations[i] := FALSE;
                        StationDelay(IN := FALSE);
                    ELSE
                        StationDelay(IN := TRUE, PT := ShutdownDelay);
                    END_IF;
                    EXIT;
                END_IF;
            END_FOR;
        END_IF;
        
        (* Complete shutdown *)
        IF FOR_ALL i := 1 TO 8 DO (NOT HeatingStations[i] AND NOT CoolingStations[i]) AND
           Feeder1_Speed = 0.0 AND Feeder2_Speed = 0.0 THEN
            MachineState := 4; (* Stopped state *)
            ShutdownTimer(IN := FALSE);
        END_IF;
        
        (* Timeout check *)
        IF ShutdownTimer.Q THEN
            FaultState := TRUE;
            MachineState := 4; (* Stopped due to fault *)
        END_IF;
    
    4: (* Stopped State *)
        (* Reset for next cycle or handle fault *)
        IF NOT EmergencyStop AND NOT FaultState AND ResetCommand THEN
            MachineState := 0; (* Return to idle *)
            (* Reset variables *)
            FOR i := 1 TO 8 DO
                HeatingStations[i] := FALSE;
                CoolingStations[i] := FALSE;
                HeatingReady[i] := FALSE;
                CoolingReady[i] := FALSE;
            END_FOR;
            Feeder1_Speed := 0.0;
            Feeder2_Speed := 0.0;
            HorizontalCutter := FALSE;
            VerticalCutter := FALSE;
            CutterSyncPulse := FALSE;
        END_IF;
END_CASE;

(* Safety Interlocks *)
IF EmergencyStop OR FaultState THEN
    (* Immediate shutdown of all systems *)
    FOR i := 1 TO 8 DO
        HeatingStations[i] := FALSE;
        CoolingStations[i] := FALSE;
    END_FOR;
    Feeder1_Speed := 0.0;
    Feeder2_Speed := 0.0;
    HorizontalCutter := FALSE;
    VerticalCutter := FALSE;
    MachineState := 4; (* Stopped state *)
END_IF;

(* Temperature Monitoring for Faults *)
FOR i := 1 TO 8 DO
    IF HeatingTemps[i] > (HeatingSetpoint + 20.0) OR
       CoolingTemps[i] < (CoolingSetpoint - 20.0) THEN
        FaultState := TRUE;
    END_IF;
END_FOR;

(* Tension Monitoring for Faults *)
IF ABS(TensionSensor1 - TensionSetpoint) > (TensionTolerance * 2.0) OR
   ABS(TensionSensor2 - TensionSetpoint) > (TensionTolerance * 2.0) THEN
    FaultState := TRUE;
END_IF;

END_PROGRAM
