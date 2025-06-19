(* IEC 61131-3 Structured Text program for 3D pouch making machine start-up and shutdown *)
(* Manages 8 heating stations, 8 cooling stations, horizontal/vertical cutters, and two feeder units *)

PROGRAM PouchMachineControl
VAR
    (* Machine state variables *)
    MachineState : INT := 0; (* 0=Stopped, 1=Starting, 2=Running, 3=Stopping *)
    StartCommand : BOOL := FALSE; (* Operator start signal *)
    StopCommand : BOOL := FALSE; (* Operator stop signal *)
    EmergencyStop : BOOL := FALSE; (* Emergency stop signal *)
    
    (* Heating station variables *)
    HeatingStationOn : ARRAY[1..8] OF BOOL; (* Status of each heating station *)
    HeatingTemp : ARRAY[1..8] OF REAL; (* Current temperature of each station *)
    TARGET_HEAT_TEMP : REAL := 180.0; (* Target temperature in Celsius *)
    HEAT_TEMP_TOLERANCE : REAL := 5.0; (* Acceptable temperature deviation *)
    
    (* Cooling station variables *)
    CoolingStationOn : ARRAY[1..8] OF BOOL; (* Status of each cooling station *)
    CoolingTemp : ARRAY[1..8] OF REAL; (* Current temperature of each station *)
    TARGET_COOL_TEMP : REAL := 25.0; (* Target cooling temperature in Celsius *)
    COOL_TEMP_TOLERANCE : REAL := 3.0; (* Acceptable temperature deviation *)
    
    (* Feeder unit variables *)
    Feeder1_Speed : REAL := 0.0; (* Feeder 1 speed in m/min *)
    Feeder2_Speed : REAL := 0.0; (* Feeder 2 speed in m/min *)
    TensionSetpoint : REAL := 10.0; (* Target winding tension in N *)
    TensionActual : REAL; (* Measured tension from sensor *)
    TENSION_TOLERANCE : REAL := 0.5; (* Acceptable tension deviation *)
    
    (* Cutter variables *)
    HorizontalCutterActive : BOOL := FALSE; (* Horizontal cutter status *)
    VerticalCutterActive : BOOL := FALSE; (* Vertical cutter status *)
    MaterialFlowRate : REAL := 0.0; (* Material flow rate in m/min *)
    CUTTER_SYNC_THRESHOLD : REAL := 0.1; (* Max flow rate deviation for sync *)
    
    (* Timers *)
    HeatingDelay : TON; (* Timer for heating station warm-up *)
    CoolingDelay : TON; (* Timer for cooling station stabilization *)
    TensionStabilizeDelay : TON; (* Timer for tension stabilization *)
    CutterSyncDelay : TON; (* Timer for cutter synchronization *)
    ShutdownCoolingDelay : TON; (* Timer for cooling during shutdown *)
    
    (* Timer presets *)
    HEATING_DELAY_TIME : TIME := T#10s; (* Time to reach target heat temp *)
    COOLING_DELAY_TIME : TIME := T#5s; (* Time to stabilize cooling *)
    TENSION_DELAY_TIME : TIME := T#3s; (* Time to stabilize tension *)
    CUTTER_SYNC_TIME : TIME := T#2s; (* Time to synchronize cutters *)
    SHUTDOWN_COOLING_TIME : TIME := T#30s; (* Time for shutdown cooling *)
    
    (* Fault flags *)
    HeatingFault : BOOL := FALSE; (* Fault in heating stations *)
    CoolingFault : BOOL := FALSE; (* Fault in cooling stations *)
    TensionFault : BOOL := FALSE; (* Fault in tension control *)
    CutterSyncFault : BOOL := FALSE; (* Fault in cutter synchronization *)
END_VAR

(* Main control logic *)
CASE MachineState OF
    0: (* Stopped state *)
        IF StartCommand AND NOT EmergencyStop THEN
            (* Initiate start-up sequence *)
            MachineState := 1;
            StartCommand := FALSE;
            (* Reset timers *)
            HeatingDelay(IN := FALSE);
            CoolingDelay(IN := FALSE);
            TensionStabilizeDelay(IN := FALSE);
            CutterSyncDelay(IN := FALSE);
        END_IF;
        
    1: (* Starting state *)
        (* Step 1: Activate heating stations sequentially *)
        FOR i := 1 TO 8 DO
            IF NOT HeatingStationOn[i] THEN
                HeatingStationOn[i] := TRUE; (* Turn on heating station *)
                HeatingDelay(IN := TRUE, PT := HEATING_DELAY_TIME);
                EXIT; (* Wait for this station to heat up *)
            END_IF;
        END_FOR;
        
        (* Check heating station temperatures *)
        IF HeatingDelay.Q THEN
            FOR i := 1 TO 8 DO
                IF HeatingStationOn[i] AND (ABS(HeatingTemp[i] - TARGET_HEAT_TEMP) > HEAT_TEMP_TOLERANCE) THEN
                    HeatingFault := TRUE; (* Set fault if temp out of range *)
                    MachineState := 0; (* Return to stopped state *)
                    EXIT;
                END_IF;
            END_FOR;
            HeatingDelay(IN := FALSE); (* Reset timer *)
        END_IF;
        
        (* Step 2: Activate cooling stations *)
        IF NOT HeatingFault AND NOT HeatingDelay.IN THEN
            FOR i := 1 TO 8 DO
                IF NOT CoolingStationOn[i] THEN
                    CoolingStationOn[i] := TRUE; (* Turn on cooling station *)
                    CoolingDelay(IN := TRUE, PT := COOLING_DELAY_TIME);
                    EXIT;
                END_IF;
            END_FOR;
        END_IF;
        
        (* Check cooling station temperatures *)
        IF CoolingDelay.Q THEN
            FOR i := 1 TO 8 DO
                IF CoolingStationOn[i] AND (ABS(CoolingTemp[i] - TARGET_COOL_TEMP) > COOL_TEMP_TOLERANCE) THEN
                    CoolingFault := TRUE; (* Set fault if temp out of range *)
                    MachineState := 0;
                    EXIT;
                END_IF;
            END_FOR;
            CoolingDelay(IN := FALSE);
        END_IF;
        
        (* Step 3: Start feeder units and control tension *)
        IF NOT HeatingFault AND NOT CoolingFault AND NOT CoolingDelay.IN THEN
            Feeder1_Speed := 5.0; (* Initial feeder speed *)
            Feeder2_Speed := 5.0;
            TensionStabilizeDelay(IN := TRUE, PT := TENSION_DELAY_TIME);
            
            (* Adjust feeder speeds to maintain tension *)
            IF ABS(TensionActual - TensionSetpoint) > TENSION_TOLERANCE THEN
                IF TensionActual < TensionSetpoint THEN
                    Feeder2_Speed := Feeder2_Speed + 0.1; (* Increase downstream speed *)
                ELSE
                    Feeder2_Speed := Feeder2_Speed - 0.1; (* Decrease downstream speed *)
                END_IF;
            END_IF;
            
            IF TensionStabilizeDelay.Q AND (ABS(TensionActual - TensionSetpoint) > TENSION_TOLERANCE) THEN
                TensionFault := TRUE;
                MachineState := 0;
            ELSIF TensionStabilizeDelay.Q THEN
                TensionStabilizeDelay(IN := FALSE);
            END_IF;
        END_IF;
        
        (* Step 4: Synchronize cutters with material flow *)
        IF NOT TensionFault AND NOT TensionStabilizeDelay.IN THEN
            HorizontalCutterActive := TRUE;
            VerticalCutterActive := TRUE;
            CutterSyncDelay(IN := TRUE, PT := CUTTER_SYNC_TIME);
            
            (* Check synchronization *)
            IF ABS(MaterialFlowRate - Feeder1_Speed) > CUTTER_SYNC_THRESHOLD THEN
                CutterSyncFault := TRUE;
                MachineState := 0;
            ELSIF CutterSyncDelay.Q THEN
                CutterSyncDelay(IN := FALSE);
                MachineState := 2; (* Transition to running state *)
            END_IF;
        END_IF;
        
    2: (* Running state *)
        (* Maintain tension control *)
        IF ABS(TensionActual - TensionSetpoint) > TENSION_TOLERANCE THEN
            IF TensionActual < TensionSetpoint THEN
                Feeder2_Speed := Feeder2_Speed + 0.1;
            ELSE
                Feeder2_Speed := Feeder2_Speed - 0.1;
            END_IF;
        END_IF;
        
        (* Monitor for stop command or faults *)
        IF StopCommand OR EmergencyStop OR HeatingFault OR CoolingFault OR TensionFault OR CutterSyncFault THEN
            MachineState := 3; (* Initiate shutdown *)
            StopCommand := FALSE;
        END_IF;
        
    3: (* Stopping state *)
        (* Step 1: Deactivate cutters *)
        HorizontalCutterActive := FALSE;
        VerticalCutterActive := FALSE;
        
        (* Step 2: Ramp down feeder speeds *)
        Feeder1_Speed := Feeder1_Speed - 0.2; (* Gradual slowdown *)
        Feeder2_Speed := Feeder2_Speed - 0.2;
        IF Feeder1_Speed <= 0.0 AND Feeder2_Speed <= 0.0 THEN
            Feeder1_Speed := 0.0;
            Feeder2_Speed := 0.0;
        END_IF;
        
        (* Step 3: Deactivate heating stations *)
        IF Feeder1_Speed = 0.0 THEN
            FOR i := 1 TO 8 DO
                HeatingStationOn[i] := FALSE;
            END_FOR;
        END_IF;
        
        (* Step 4: Allow cooling stations to stabilize *)
        IF NOT HeatingStationOn[1] THEN (* All heaters off *)
            ShutdownCoolingDelay(IN := TRUE, PT := SHUTDOWN_COOLING_TIME);
            FOR i := 1 TO 8 DO
                IF CoolingTemp[i] > TARGET_COOL_TEMP + COOL_TEMP_TOLERANCE THEN
                    CoolingStationOn[i] := TRUE; (* Keep cooling active *)
                ELSE
                    CoolingStationOn[i] := FALSE;
                END_IF;
            END_FOR;
        END_IF;
        
        (* Step 5: Complete shutdown *)
        IF ShutdownCoolingDelay.Q THEN
            FOR i := 1 TO 8 DO
                CoolingStationOn[i] := FALSE; (* Turn off all cooling *)
            END_FOR;
            ShutdownCoolingDelay(IN := FALSE);
            MachineState := 0; (* Return to stopped state *)
        END_IF;
END_CASE;

(* Reset faults on emergency stop *)
IF EmergencyStop THEN
    MachineState := 0;
    FOR i := 1 TO 8 DO
        HeatingStationOn[i] := FALSE;
        CoolingStationOn[i] := FALSE;
    END_FOR;
    Feeder1_Speed := 0.0;
    Feeder2_Speed := 0.0;
    HorizontalCutterActive := FALSE;
    VerticalCutterActive := FALSE;
    HeatingFault := FALSE;
    CoolingFault := FALSE;
    TensionFault := FALSE;
    CutterSyncFault := FALSE;
END_IF;

END_PROGRAM
