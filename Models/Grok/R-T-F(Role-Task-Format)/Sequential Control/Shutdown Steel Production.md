(* IEC 61131-3 Structured Text program for safe furnace shutdown in steel production *)
(* Implements phased shutdown with gas flow reduction and oxygen adjustment *)

PROGRAM SteelFurnaceShutdown
VAR
    (* Shutdown control variables *)
    ShutdownPhase : INT := 0; (* 0=Idle, 1=ProdHalt, 2=InitCooldown, 3=GasInit, 4=GasReduction, 5=OxygenAdjust, 6=FinalCooldown, 7=Isolate, 8=Complete *)
    StartShutdown : BOOL := FALSE; (* Command to start shutdown *)
    ShutdownComplete : BOOL := FALSE; (* Flag for shutdown completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for safety issues *)
    EmergencyStop : BOOL := FALSE; (* Emergency stop signal *)
    
    (* Process parameters *)
    InitialGasFlow : REAL := 1000.0; (* Initial fuel gas flow in Nm³/h *)
    TargetGasFlow : REAL := 0.0; (* Final gas flow *)
    GasRampTime : TIME := T#12h; (* 12-hour gas flow ramp-down *)
    FuelToAirRatio : REAL := 2.5; (* Fuel-to-air ratio (1:2.5) *)
    OxygenFractionAir : REAL := 0.21; (* Oxygen fraction in air *)
    TargetOxygenFlow : REAL; (* Calculated oxygen flow in Nm³/h *)
    InitialTemp : REAL := 1500.0; (* Initial furnace temperature in Celsius *)
    InitCooldownTemp : REAL := 1000.0; (* Initial cooldown target *)
    FinalCooldownTemp : REAL := 200.0; (* Final cooldown target *)
    MinCombustionTemp : REAL := 500.0; (* Minimum temperature for combustion *)
    CoolingWaterFlow : REAL := 500.0; (* Cooling water flow in m³/h *)
    NitrogenFlow : REAL := 200.0; (* Nitrogen purge flow in Nm³/h *)
    MaxExhaustOxygen : REAL := 5.0; (* Max exhaust oxygen concentration in % *)
    MaxGasPressure : REAL := 5.0; (* Max gas pressure in bar *)
    MaxWaterTemp : REAL := 80.0; (* Max cooling water temperature *)
    MaxFurnacePressure : REAL := 0.1; (* Max furnace pressure in bar *)
    
    (* Input variables *)
    CurrentTemp : REAL; (* Current furnace temperature *)
    CurrentGasFlow : REAL; (* Current fuel gas flow *)
    CurrentOxygenFlow : REAL; (* Current oxygen flow *)
    CurrentExhaustOxygen : REAL; (* Current exhaust oxygen concentration *)
    CurrentGasPressure : REAL; (* Current gas pressure *)
    CurrentWaterTemp : REAL; (* Current cooling water temperature *)
    CurrentFurnacePressure : REAL; (* Current furnace pressure *)
    FlameSensor : BOOL; (* Flame presence feedback *)
    GasValveFeedback : BOOL; (* Gas valve operational status *)
    OxygenValveFeedback : BOOL; (* Oxygen valve operational status *)
    CoolingSystemFeedback : BOOL; (* Cooling system status *)
    NitrogenValveFeedback : BOOL; (* Nitrogen valve status *)
    
    (* Output variables *)
    GasValveCommand : REAL; (* Command to gas valve (0–1000 Nm³/h) *)
    OxygenValveCommand : REAL; (* Command to oxygen valve (0–525 Nm³/h) *)
    CoolingWaterCommand : BOOL; (* Command to cooling system *)
    NitrogenValveCommand : BOOL; (* Command to nitrogen valve *)
    PowerIsolateCommand : BOOL; (* Command to isolate power *)
    
    (* Timers *)
    InitCooldownTimer : TON; (* Timer for initial cooldown *)
    GasReductionTimer : TON; (* Timer for gas flow reduction *)
    OxygenAdjustTimer : TON; (* Timer for oxygen adjustment interval *)
    FinalCooldownTimer : TON; (* Timer for final cooldown *)
    
    (* Timer presets *)
    INIT_COOLDOWN_TIME : TIME := T#4h; (* 4-hour initial cooldown *)
    OXYGEN_ADJUST_INTERVAL : TIME := T#10s; (* 10-second oxygen adjustment interval *)
    FINAL_COOLDOWN_TIME : TIME := T#8h; (* 8-hour final cooldown *)
END_VAR

(* Function block for ramping down fuel gas flow *)
FUNCTION_BLOCK RampDownGasFlow
    VAR_INPUT
        InitialFlow : REAL; (* Starting gas flow in Nm³/h *)
        TargetFlow : REAL; (* Target gas flow *)
        RampTime : TIME; (* Total ramp duration *)
        CurrentTime : TIME; (* Elapsed time from timer *)
        FurnaceTemp : REAL; (* Current furnace temperature *)
        MinTemp : REAL; (* Minimum combustion temperature *)
        FlameStatus : BOOL; (* Flame sensor status *)
        ValveFeedback : BOOL; (* Gas valve feedback *)
        MaxPressure : REAL; (* Maximum gas pressure *)
        GasPressure : REAL; (* Current gas pressure *)
    END_VAR
    VAR_OUTPUT
        GasFlowCommand : REAL; (* Commanded gas flow *)
        RampComplete : BOOL; (* Ramp completion flag *)
        Fault : BOOL; (* Fault flag *)
    END_VAR
    VAR
        RampRate : REAL; (* Flow reduction rate in Nm³/h per second *)
        TimeElapsed : REAL; (* Elapsed time in seconds *)
        TotalTime : REAL; (* Total ramp time in seconds *)
    END_VAR
    (* Calculate ramp rate *)
    TotalTime := TIME_TO_REAL(RampTime) / 1000.0; (* Convert ms to seconds *)
    TimeElapsed := TIME_TO_REAL(CurrentTime) / 1000.0;
    RampRate := (InitialFlow - TargetFlow) / TotalTime; (* Nm³/h per second *)
    
    (* Safety checks *)
    IF FurnaceTemp < MinTemp OR NOT FlameStatus OR GasPressure > MaxPressure OR NOT ValveFeedback THEN
        Fault := TRUE;
        GasFlowCommand := 0.0;
    ELSE
        (* Calculate commanded flow *)
        GasFlowCommand := InitialFlow - (RampRate * TimeElapsed);
        IF GasFlowCommand <= TargetFlow THEN
            GasFlowCommand := TargetFlow;
            RampComplete := TRUE;
        END_IF;
    END_IF;
END_FUNCTION_BLOCK

(* Function block for adjusting oxygen supply *)
FUNCTION_BLOCK AdjustOxygenSupply
    VAR_INPUT
        GasFlow : REAL; (* Current gas flow in Nm³/h *)
        FurnaceTemp : REAL; (* Current furnace temperature *)
        MinTemp : REAL; (* Minimum combustion temperature *)
        FuelToAirRatio : REAL; (* Target fuel-to-air ratio *)
        OxygenFraction : REAL; (* Oxygen fraction in air *)
        ExhaustOxygen : REAL; (* Exhaust oxygen concentration *)
        MaxExhaustOxygen : REAL; (* Max allowable exhaust oxygen *)
        FlameStatus : BOOL; (* Flame sensor status *)
        ValveFeedback : BOOL; (* Oxygen valve feedback *)
    END_VAR
    VAR_OUTPUT
        OxygenFlowCommand : REAL; (* Commanded oxygen flow *)
        Fault : BOOL; (* Fault flag *)
    END_VAR
    (* Calculate required oxygen flow *)
    IF FurnaceTemp < MinTemp OR NOT FlameStatus OR ExhaustOxygen > MaxExhaustOxygen OR NOT ValveFeedback THEN
        Fault := TRUE;
        OxygenFlowCommand := 0.0;
    ELSE
        (* Oxygen flow = GasFlow * FuelToAirRatio * OxygenFraction *)
        OxygenFlowCommand := GasFlow * FuelToAirRatio * OxygenFraction;
    END_IF;
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    GasRampFB : RampDownGasFlow;
    OxygenAdjustFB : AdjustOxygenSupply;
END_VAR

(* Main shutdown sequence logic *)
CASE ShutdownPhase OF
    0: (* Idle - Waiting for shutdown command *)
        IF StartShutdown AND NOT EmergencyStop THEN
            ShutdownPhase := 1; (* Start production halt *)
            StartShutdown := FALSE;
            (* Reset timers *)
            InitCooldownTimer(IN := FALSE);
            GasReductionTimer(IN := FALSE);
            OxygenAdjustTimer(IN := FALSE);
            FinalCooldownTimer(IN := FALSE);
        END_IF;
        
    1: (* Production Halt - Stop material feed *)
        (* Assume material feed stopped externally *)
        ShutdownPhase := 2; (* Transition to initial cooldown *)
        
    2: (* Initial Cooldown - Reduce to 1000°C *)
        IF CurrentTemp <= InitCooldownTemp THEN
            InitCooldownTimer(IN := FALSE);
            ShutdownPhase := 3; (* Transition to gas flow initiation *)
        ELSE
            InitCooldownTimer(IN := TRUE, PT := INIT_COOLDOWN_TIME);
            (* Reduce gas flow slightly to initiate cooldown *)
            GasValveCommand := InitialGasFlow * 0.8; (* 80% flow *)
            IF InitCooldownTimer.Q THEN
                FaultDetected := TRUE;
                ShutdownPhase := 0;
            END_IF;
        END_IF;
        
    3: (* Gas Flow Initiation - Prepare for reduction *)
        GasReductionTimer(IN := TRUE, PT := GasRampTime);
        ShutdownPhase := 4; (* Start gas flow reduction *)
        
    4: (* Gas Flow Reduction - Ramp down over 12 hours *)
        GasRampFB(InitialFlow := InitialGasFlow, 
                 TargetFlow := TargetGasFlow, 
                 RampTime := GasRampTime, 
                 CurrentTime := GasReductionTimer.ET, 
                 FurnaceTemp := CurrentTemp, 
                 MinTemp := MinCombustionTemp, 
                 FlameStatus := FlameSensor, 
                 ValveFeedback := GasValveFeedback, 
                 MaxPressure := MaxGasPressure, 
                 GasPressure := CurrentGasPressure);
        GasValveCommand := GasRampFB.GasFlowCommand;
        
        IF GasRampFB.Fault THEN
            FaultDetected := TRUE;
            ShutdownPhase := 0;
        ELSIF GasRampFB.RampComplete THEN
            GasReductionTimer(IN := FALSE);
            ShutdownPhase := 5; (* Transition to oxygen adjustment *)
        END_IF;
        
    5: (* Oxygen Level Adjustment - Maintain 1:2.5 ratio *)
        OxygenAdjustTimer(IN := TRUE, PT := OXYGEN_ADJUST_INTERVAL);
        IF OxygenAdjustTimer.Q THEN
            OxygenAdjustFB(GasFlow := CurrentGasFlow, 
                          FurnaceTemp := CurrentTemp, 
                          MinTemp := MinCombustionTemp, 
                          FuelToAirRatio := FuelToAirRatio, 
                          OxygenFraction := OxygenFractionAir, 
                          ExhaustOxygen := CurrentExhaustOxygen, 
                          MaxExhaustOxygen := MaxExhaustOxygen, 
                          FlameStatus := FlameSensor, 
                          ValveFeedback := OxygenValveFeedback);
            OxygenValveCommand := OxygenAdjustFB.OxygenFlowCommand;
            OxygenAdjustTimer(IN := FALSE); (* Reset for next adjustment *)
            
            IF OxygenAdjustFB.Fault THEN
                FaultDetected := TRUE;
                ShutdownPhase := 0;
            END_IF;
        END_IF;
        
        (* Run concurrently with gas reduction; check if gas flow is complete *)
        IF GasRampFB.RampComplete AND CurrentGasFlow <= TargetGasFlow THEN
            ShutdownPhase := 6; (* Transition to final cooldown *)
        END_IF;
        
    6: (* Final Cooldown - Cool to <200°C *)
        GasValveCommand := 0.0; (* Ensure gas flow off *)
        OxygenValveCommand := 0.0; (* Ensure oxygen flow off *)
        CoolingWaterCommand := TRUE; (* Activate cooling system *)
        NitrogenValveCommand := TRUE; (* Start nitrogen purge *)
        FinalCooldownTimer(IN := TRUE, PT := FINAL_COOLDOWN_TIME);
        
        (* Safety checks *)
        IF CurrentWaterTemp > MaxWaterTemp OR CurrentFurnacePressure > MaxFurnacePressure OR NOT CoolingSystemFeedback OR NOT NitrogenValveFeedback THEN
            FaultDetected := TRUE;
            ShutdownPhase := 0;
        ELSIF CurrentTemp <= FinalCooldownTemp THEN
            FinalCooldownTimer(IN := FALSE);
            ShutdownPhase := 7; (* Transition to system isolation *)
        END_IF;
        
    7: (* System Isolation - Shut off utilities *)
        CoolingWaterCommand := FALSE;
        NitrogenValveCommand := FALSE;
        PowerIsolateCommand := TRUE; (* Isolate power *)
        IF CurrentFurnacePressure <= 0.1 AND NOT PowerIsolateCommand THEN
            ShutdownComplete := TRUE;
            ShutdownPhase := 8; (* Transition to complete *)
        END_IF;
        
    8: (* Complete - Shutdown finished *)
        (* All outputs off; await reset *)
        GasValveCommand := 0.0;
        OxygenValveCommand := 0.0;
        CoolingWaterCommand := FALSE;
        NitrogenValveCommand := FALSE;
        PowerIsolateCommand := FALSE;
END_CASE;

(* Emergency stop and fault handling *)
IF EmergencyStop OR FaultDetected THEN
    ShutdownPhase := 0;
    GasValveCommand := 0.0;
    OxygenValveCommand := 0.0;
    CoolingWaterCommand := FALSE;
    NitrogenValveCommand := FALSE;
    PowerIsolateCommand := TRUE;
    (* Fault or stop must be acknowledged *)
END_IF;

END_PROGRAM
