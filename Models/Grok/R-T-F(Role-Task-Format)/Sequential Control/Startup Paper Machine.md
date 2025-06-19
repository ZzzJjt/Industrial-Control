(* IEC 61131-3 Structured Text program for Valmet paper machine press section startup *)
(* Implements sequential startup with speed, nip pressure, and temperature control *)
(* Ensures safe transitions and stable operation to prevent sheet breakage *)

PROGRAM PaperMachinePressStartup
VAR
    (* Startup control variables *)
    StartupPhase : INT := 0; (* 0=Idle, 1=PreStart, 2=ConveyorStart, 3=RollStart, 4=TempStabilize, 5=NipPressure, 6=SpeedRamp, 7=SteadyState *)
    StartCommand : BOOL := FALSE; (* Operator start signal *)
    StartupComplete : BOOL := FALSE; (* Flag for startup completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for safety issues *)
    EmergencyStop : BOOL := FALSE; (* Emergency stop signal *)
    
    (* Process parameters *)
    TargetRollSpeed : REAL := 500.0; (* Target press roll speed in m/min *)
    InitialRollSpeed : REAL := 50.0; (* Initial roll speed *)
    SpeedRampTime : TIME := T#10m; (* 10-minute speed ramp *)
    TargetNipPressure : REAL := 250.0; (* Target nip pressure in kN/m *)
    NipRampTime : TIME := T#5m; (* 5-minute pressure ramp per roll pair *)
    TargetFeltTemp : REAL := 87.5; (* Target felt temperature in Celsius *)
    TempTolerance : REAL := 2.5; (* Acceptable temperature deviation *)
    MinWebTension : REAL := 1.0; (* Minimum web tension in kN *)
    MaxHydraulicPressure : REAL := 300.0; (* Max hydraulic pressure in kN/m *)
    
    (* Input variables *)
    CurrentRollSpeed : REAL; (* Current press roll speed *)
    CurrentNipPressure1 : REAL; (* Current nip pressure for first roll pair *)
    CurrentNipPressure2 : REAL; (* Current nip pressure for second roll pair *)
    CurrentNipPressure3 : REAL; (* Current nip pressure for third roll pair *)
    CurrentFeltTemp : REAL; (* Current felt temperature *)
    CurrentWebTension : REAL; (* Current web tension *)
    ConveyorFeedback : BOOL; (* Conveyor operational status *)
    RollFeedback : BOOL; (* Roll operational status *)
    HydraulicFeedback : BOOL; (* Hydraulic system status *)
    SteamValveFeedback : BOOL; (* Steam valve status *)
    SafetyGuards : BOOL; (* Safety guards in place *)
    
    (* Output variables *)
    ConveyorCommand : BOOL; (* Command to conveyors *)
    RollCommand1 : BOOL; (* Command to first roll pair *)
    RollCommand2 : BOOL; (* Command to second roll pair *)
    RollCommand3 : BOOL; (* Command to third roll pair *)
    SpeedCommand : REAL; (* Commanded roll/conveyor speed *)
    NipPressureCommand1 : REAL; (* Commanded nip pressure for first roll pair *)
    NipPressureCommand2 : REAL; (* Commanded nip pressure for second roll pair *)
    NipPressureCommand3 : REAL; (* Commanded nip pressure for third roll pair *)
    SteamValveCommand : BOOL; (* Command to steam valve *)
    
    (* Timers *)
    ConveyorStartTimer : TON; (* Timer for conveyor startup *)
    RollStartTimer : TON; (* Timer for roll startup *)
    TempStabilizeTimer : TON; (* Timer for temperature stabilization *)
    NipPressureTimer : TON; (* Timer for nip pressure ramp *)
    SpeedRampTimer : TON; (* Timer for speed ramp *)
    
    (* Timer presets *)
    CONVEYOR_START_TIME : TIME := T#30s; (* Time to stabilize conveyors *)
    ROLL_START_TIME : TIME := T#1m; (* Time to engage each roll pair *)
    TEMP_STABILIZE_TIME : TIME := T#5m; (* Max time to reach felt temperature *)
    STEADY_STATE_CHECK_TIME : TIME := T#2m; (* Time to verify steady state *)
END_VAR

(* Function block for ramping speed *)
FUNCTION_BLOCK RampSpeed
    VAR_INPUT
        InitialSpeed : REAL; (* Starting speed in m/min *)
        TargetSpeed : REAL; (* Target speed *)
        RampTime : TIME; (* Total ramp duration *)
        CurrentTime : TIME; (* Elapsed time from timer *)
        Feedback : BOOL; (* Equipment operational status *)
    END_VAR
    VAR_OUTPUT
        SpeedCommand : REAL; (* Commanded speed *)
        RampComplete : BOOL; (* Ramp completion flag *)
        Fault : BOOL; (* Fault flag *)
    END_VAR
    VAR
        RampRate : REAL; (* Speed increase rate in m/min per second *)
        TimeElapsed : REAL; (* Elapsed time in seconds *)
        TotalTime : REAL; (* Total ramp time in seconds *)
    END_VAR
    (* Calculate ramp rate *)
    TotalTime := TIME_TO_REAL(RampTime) / 1000.0; (* Convert ms to seconds *)
    TimeElapsed := TIME_TO_REAL(CurrentTime) / 1000.0;
    RampRate := (TargetSpeed - InitialSpeed) / TotalTime; (* m/min per second *)
    
    (* Safety check *)
    IF NOT Feedback THEN
        Fault := TRUE;
        SpeedCommand := 0.0;
    ELSE
        (* Calculate commanded speed *)
        SpeedCommand := InitialSpeed + (RampRate * TimeElapsed);
        IF SpeedCommand >= TargetSpeed THEN
            SpeedCommand := TargetSpeed;
            RampComplete := TRUE;
        END_IF;
    END_IF;
END_FUNCTION_BLOCK

(* Function block for ramping nip pressure *)
FUNCTION_BLOCK RampNipPressure
    VAR_INPUT
        InitialPressure : REAL; (* Starting pressure in kN/m *)
        TargetPressure : REAL; (* Target pressure *)
        RampTime : TIME; (* Total ramp duration *)
        CurrentTime : TIME; (* Elapsed time from timer *)
        HydraulicFeedback : BOOL; (* Hydraulic system status *)
        MaxPressure : REAL; (* Max allowable pressure *)
        CurrentPressure : REAL; (* Current pressure *)
    END_VAR
    VAR_OUTPUT
        PressureCommand : REAL; (* Commanded pressure *)
        RampComplete : BOOL; (* Ramp completion flag *)
        Fault : BOOL; (* Fault flag *)
    END_VAR
    VAR
        RampRate : REAL; (* Pressure increase rate in kN/m per second *)
        TimeElapsed : REAL; (* Elapsed time in seconds *)
        TotalTime : REAL; (* Total ramp time in seconds *)
    END_VAR
    (* Calculate ramp rate *)
    TotalTime := TIME_TO_REAL(RampTime) / 1000.0;
    TimeElapsed := TIME_TO_REAL(CurrentTime) / 1000.0;
    RampRate := (TargetPressure - InitialPressure) / TotalTime;
    
    (* Safety checks *)
    IF NOT HydraulicFeedback OR CurrentPressure > MaxPressure THEN
        Fault := TRUE;
        PressureCommand := 0.0;
    ELSE
        (* Calculate commanded pressure *)
        PressureCommand := InitialPressure + (RampRate * TimeElapsed);
        IF PressureCommand >= TargetPressure THEN
            PressureCommand := TargetPressure;
            RampComplete := TRUE;
        END_IF;
    END_IF;
END_FUNCTION_BLOCK

(* Function block for temperature stabilization *)
FUNCTION_BLOCK StabilizeTemperature
    VAR_INPUT
        TargetTemp : REAL; (* Target temperature in Celsius *)
        CurrentTemp : REAL; (* Current temperature *)
        Tolerance : REAL; (* Acceptable deviation *)
        ValveFeedback : BOOL; (* Steam valve status *)
    END_VAR
    VAR_OUTPUT
        SteamValveOn : BOOL; (* Command to steam valve *)
        TempStable : BOOL; (* Temperature stability flag *)
        Fault : BOOL; (* Fault flag *)
    END_VAR
    (* Control steam valve *)
    IF NOT ValveFeedback THEN
        Fault := TRUE;
        SteamValveOn := FALSE;
    ELSIF ABS(CurrentTemp - TargetTemp) <= Tolerance THEN
        TempStable := TRUE;
        SteamValveOn := FALSE;
    ELSE
        SteamValveOn := TRUE;
    END_IF;
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    SpeedRampFB : RampSpeed;
    NipPressureFB1 : RampNipPressure; (* For first roll pair *)
    NipPressureFB2 : RampNipPressure; (* For second roll pair *)
    NipPressureFB3 : RampNipPressure; (* For third roll pair *)
    TempStabilizeFB : StabilizeTemperature;
END_VAR

(* Main startup sequence logic *)
CASE StartupPhase OF
    0: (* Idle - Waiting for start command *)
        IF StartCommand AND NOT EmergencyStop THEN
            StartupPhase := 1; (* Start pre-start checks *)
            StartCommand := FALSE;
            (* Reset timers *)
            ConveyorStartTimer(IN := FALSE);
            RollStartTimer(IN := FALSE);
            TempStabilizeTimer(IN := FALSE);
            NipPressureTimer(IN := FALSE);
            SpeedRampTimer(IN := FALSE);
        END_IF;
        
    1: (* Pre-Start Checks - Verify safety conditions *)
        IF NOT SafetyGuards OR FaultDetected THEN
            FaultDetected := TRUE;
            StartupPhase := 0;
        ELSE
            StartupPhase := 2; (* Transition to conveyor activation *)
        END_IF;
        
    2: (* Conveyor Activation - Start conveyors *)
        ConveyorCommand := TRUE;
        SpeedRampFB(InitialSpeed := 0.0, 
                   TargetSpeed := InitialRollSpeed, 
                   RampTime := CONVEYOR_START_TIME, 
                   CurrentTime := ConveyorStartTimer.ET, 
                   Feedback := ConveyorFeedback);
        SpeedCommand := SpeedRampFB.SpeedCommand;
        ConveyorStartTimer(IN := TRUE, PT := CONVEYOR_START_TIME);
        
        IF SpeedRampFB.Fault OR NOT ConveyorFeedback THEN
            FaultDetected := TRUE;
            StartupPhase := 0;
        ELSIF SpeedRampFB.RampComplete THEN
            ConveyorStartTimer(IN := FALSE);
            StartupPhase := 3; (* Transition to roll activation *)
        END_IF;
        
    3: (* Roll Activation - Engage press rolls *)
        IF NOT RollCommand1 THEN
            RollCommand1 := TRUE;
            RollStartTimer(IN := TRUE, PT := ROLL_START_TIME);
        ELSIF RollCommand1 AND NOT RollCommand2 AND RollStartTimer.Q THEN
            RollCommand2 := TRUE;
            RollStartTimer(IN := TRUE, PT := ROLL_START_TIME);
        ELSIF RollCommand2 AND NOT RollCommand3 AND RollStartTimer.Q THEN
            RollCommand3 := TRUE;
            RollStartTimer(IN := TRUE, PT := ROLL_START_TIME);
        END_IF;
        
        (* Safety check *)
        IF NOT RollFeedback THEN
            FaultDetected := TRUE;
            StartupPhase := 0;
        ELSIF RollCommand3 AND RollStartTimer.Q THEN
            RollStartTimer(IN := FALSE);
            StartupPhase := 4; (* Transition to temperature stabilization *)
        END_IF;
        
    4: (* Temperature Stabilization - Heat felts to 85–90°C *)
        TempStabilizeFB(TargetTemp := TargetFeltTemp, 
                       CurrentTemp := CurrentFeltTemp, 
                       Tolerance := TempTolerance, 
                       ValveFeedback := SteamValveFeedback);
        SteamValveCommand := TempStabilizeFB.SteamValveOn;
        TempStabilizeTimer(IN := TRUE, PT := TEMP_STABILIZE_TIME);
        
        IF TempStabilizeFB.Fault OR TempStabilizeTimer.Q THEN
            FaultDetected := TRUE;
            StartupPhase := 0;
        ELSIF TempStabilizeFB.TempStable THEN
            TempStabilizeTimer(IN := FALSE);
            StartupPhase := 5; (* Transition to nip pressure ramp *)
        END_IF;
        
    5: (* Nip Pressure Ramp-Up - Ramp to 250 kN/m *)
        (* Ramp pressure for each roll pair sequentially *)
        IF NOT NipPressureFB1.RampComplete THEN
            NipPressureFB1(InitialPressure := 0.0, 
                          TargetPressure := TargetNipPressure, 
                          RampTime := NipRampTime, 
                          CurrentTime := NipPressureTimer.ET, 
                          HydraulicFeedback := HydraulicFeedback, 
                          MaxPressure := MaxHydraulicPressure, 
                          CurrentPressure := CurrentNipPressure1);
            NipPressureCommand1 := NipPressureFB1.PressureCommand;
            NipPressureTimer(IN := TRUE, PT := NipRampTime);
        ELSIF NipPressureFB1.RampComplete AND NOT NipPressureFB2.RampComplete THEN
            NipPressureFB2(InitialPressure := 0.0, 
                          TargetPressure := TargetNipPressure, 
                          RampTime := NipRampTime, 
                          CurrentTime := NipPressureTimer.ET, 
                          HydraulicFeedback := HydraulicFeedback, 
                          MaxPressure := MaxHydraulicPressure, 
                          CurrentPressure := CurrentNipPressure2);
            NipPressureCommand2 := NipPressureFB2.PressureCommand;
            NipPressureTimer(IN := TRUE, PT := NipRampTime);
        ELSIF NipPressureFB2.RampComplete AND NOT NipPressureFB3.RampComplete THEN
            NipPressureFB3(InitialPressure := 0.0, 
                          TargetPressure := TargetNipPressure, 
                          RampTime := NipRampTime, 
                          CurrentTime := NipPressureTimer.ET, 
                          HydraulicFeedback := HydraulicFeedback, 
                          MaxPressure := MaxHydraulicPressure, 
                          CurrentPressure := CurrentNipPressure3);
            NipPressureCommand3 := NipPressureFB3.PressureCommand;
            NipPressureTimer(IN := TRUE, PT := NipRampTime);
        END_IF;
        
        IF NipPressureFB1.Fault OR NipPressureFB2.Fault OR NipPressureFB3.Fault OR CurrentWebTension < MinWebTension THEN
            FaultDetected := TRUE;
            StartupPhase := 0;
        ELSIF NipPressureFB3.RampComplete THEN
            NipPressureTimer(IN := FALSE);
            StartupPhase := 6; (* Transition to speed ramp *)
        END_IF;
        
    6: (* Speed Ramp-Up - Ramp to 500 m/min *)
        SpeedRampFB(InitialSpeed := InitialRollSpeed, 
                   TargetSpeed := TargetRollSpeed, 
                   RampTime := SpeedRampTime, 
                   CurrentTime := SpeedRampTimer.ET, 
                   Feedback := RollFeedback AND ConveyorFeedback);
        SpeedCommand := SpeedRampFB.SpeedCommand;
        SpeedRampTimer(IN := TRUE, PT := SpeedRampTime);
        
        IF SpeedRampFB.Fault OR CurrentWebTension < MinWebTension THEN
            FaultDetected := TRUE;
            StartupPhase := 0;
        ELSIF SpeedRampFB.RampComplete THEN
            SpeedRampTimer(IN := FALSE);
            StartupPhase := 7; (* Transition to steady state *)
        END_IF;
        
    7: (* Steady-State Operation - Verify stability *)
        (* Maintain temperature and monitor web *)
        TempStabilizeFB(TargetTemp := TargetFeltTemp, 
                       CurrentTemp := CurrentFeltTemp, 
                       Tolerance := TempTolerance, 
                       ValveFeedback := SteamValveFeedback);
        SteamValveCommand := TempStabilizeFB.SteamValveOn;
        
        IF TempStabilizeFB.Fault OR CurrentWebTension < MinWebTension THEN
            FaultDetected := TRUE;
            StartupPhase := 0;
        ELSE
            StartupComplete := TRUE;
            (* Remain in steady state until stopped *)
        END_IF;
END_CASE;

(* Emergency stop and fault handling *)
IF EmergencyStop OR FaultDetected THEN
    StartupPhase := 0;
    ConveyorCommand := FALSE;
    RollCommand1 := FALSE;
    RollCommand2 := FALSE;
    RollCommand3 := FALSE;
    SpeedCommand := 0.0;
    NipPressureCommand1 := 0.0;
    NipPressureCommand2 := 0.0;
    NipPressureCommand3 := 0.0;
    SteamValveCommand := FALSE;
    (* Fault or stop must be acknowledged *)
END_IF;

(* Explanation of Nip Pressure and Temperature Importance *)
(* Nip Pressure: Gradual ramp-up to 250 kN/m prevents sheet breakage by avoiding sudden stress on the wet web. It also protects roll bearings and felts from overload, ensuring equipment longevity. *)
(* Temperature: Maintaining 85–90°C ensures felts absorb water effectively without sticking to the web or degrading. Stable temperature prevents thermal shock to rolls and maintains sheet quality. *)

END_PROGRAM
