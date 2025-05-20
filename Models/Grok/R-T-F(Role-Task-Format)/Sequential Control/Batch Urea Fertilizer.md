(* IEC 61131-3 Structured Text program for ISA-88-compliant urea reaction stage *)
(* Controls charging, heating, pressure regulation, reaction hold, and cooling *)

PROGRAM UreaReactionControl
VAR
    (* Batch control variables *)
    ReactionPhase : INT := 0; (* 0=Idle, 1=Charging, 2=Heating, 3=PressureReg, 4=Hold, 5=Cooling, 6=Complete *)
    StartReaction : BOOL := FALSE; (* Command to start reaction phase *)
    ReactionComplete : BOOL := FALSE; (* Flag for phase completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for process issues *)
    
    (* Process parameters *)
    TargetAmmoniaWeight : REAL := 3400.0; (* Ammonia weight in kg *)
    TargetCO2Weight : REAL := 2200.0; (* CO2 weight in kg *)
    WeightTolerance : REAL := 10.0; (* Acceptable weight deviation in kg *)
    TargetReactionTemp : REAL := 180.0; (* Reaction temperature in Celsius *)
    TempTolerance : REAL := 5.0; (* Acceptable temperature deviation *)
    TargetReactionPressure : REAL := 140.0; (* Reaction pressure in bar *)
    PressureTolerance : REAL := 5.0; (* Acceptable pressure deviation *)
    MixingSpeed : REAL := 200.0; (* Agitator speed in RPM *)
    ReactionHoldTime : TIME := T#30m; (* Reaction hold duration *)
    TargetCoolingTemp : REAL := 80.0; (* Post-reaction cooling temperature *)
    
    (* Input variables *)
    CurrentWeight : REAL; (* Current reactor weight from load cells *)
    CurrentTemp : REAL; (* Current reactor temperature *)
    CurrentPressure : REAL; (* Current reactor pressure *)
    AgitatorFeedback : BOOL; (* Agitator operational status *)
    ValveFeedbackAmmonia : BOOL; (* Ammonia valve status *)
    ValveFeedbackCO2 : BOOL; (* CO2 valve status *)
    
    (* Output variables *)
    AmmoniaValveCommand : BOOL; (* Command to ammonia valve *)
    CO2ValveCommand : BOOL; (* Command to CO2 valve *)
    HeaterCommand : BOOL; (* Command to heater *)
    CoolerCommand : BOOL; (* Command to cooling system *)
    AgitatorCommand : BOOL; (* Command to agitator *)
    PressureValveCommand : BOOL; (* Command to pressure control valve *)
    
    (* Timers *)
    ChargingTimer : TON; (* Timer for material charging *)
    HeatingTimer : TON; (* Timer for heating *)
    PressureTimer : TON; (* Timer for pressure stabilization *)
    HoldTimer : TON; (* Timer for reaction hold *)
    CoolingTimer : TON; (* Timer for cooling *)
    
    (* Timer presets *)
    CHARGING_TIMEOUT : TIME := T#10m; (* Max time to charge materials *)
    HEATING_TIMEOUT : TIME := T#15m; (* Max time to reach reaction temperature *)
    PRESSURE_TIMEOUT : TIME := T#5m; (* Max time to stabilize pressure *)
    COOLING_TIMEOUT : TIME := T#20m; (* Max time to reach cooling temperature *)
END_VAR

(* Modular function block declarations *)
FUNCTION_BLOCK AddMaterial
    VAR_INPUT
        TargetWeight : REAL;
        CurrentWeight : REAL;
        ValveFeedback : BOOL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        ValveOn : BOOL;
        AdditionDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control material valve and verify weight *)
    IF ABS(CurrentWeight - TargetWeight) <= Tolerance THEN
        AdditionDone := TRUE;
        ValveOn := FALSE;
    ELSIF ValveFeedback THEN
        ValveOn := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartHeating
    VAR_INPUT
        TargetTemp : REAL;
        CurrentTemp : REAL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        HeaterOn : BOOL;
        HeatingDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control heater and verify temperature *)
    IF ABS(CurrentTemp - TargetTemp) <= Tolerance THEN
        HeatingDone := TRUE;
        HeaterOn := FALSE;
    ELSE
        HeaterOn := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK RegulatePressure
    VAR_INPUT
        TargetPressure : REAL;
        CurrentPressure : REAL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        ValveOn : BOOL;
        PressureStable : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control pressure valve *)
    IF ABS(CurrentPressure - TargetPressure) <= Tolerance THEN
        PressureStable := TRUE;
        ValveOn := FALSE;
    ELSE
        ValveOn := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartMixing
    VAR_INPUT
        TargetSpeed : REAL;
        Feedback : BOOL;
    END_VAR
    VAR_OUTPUT
        MixerOn : BOOL;
        MixingStable : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control agitator and verify operation *)
    IF Feedback THEN
        MixerOn := TRUE;
        MixingStable := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartCooling
    VAR_INPUT
        TargetTemp : REAL;
        CurrentTemp : REAL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        CoolerOn : BOOL;
        CoolingDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control cooling system *)
    IF CurrentTemp <= TargetTemp + Tolerance THEN
        CoolingDone := TRUE;
        CoolerOn := FALSE;
    ELSE
        CoolerOn := TRUE;
    END_IF;
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    AmmoniaAddFB : AddMaterial;
    CO2AddFB : AddMaterial;
    HeatingFB : StartHeating;
    PressureFB : RegulatePressure;
    MixingFB : StartMixing;
    CoolingFB : StartCooling;
END_VAR

(* Main reaction stage logic *)
CASE ReactionPhase OF
    0: (* Idle - Waiting for start command *)
        IF StartReaction THEN
            ReactionPhase := 1; (* Start charging phase *)
            StartReaction := FALSE;
            (* Reset timers *)
            ChargingTimer(IN := FALSE);
            HeatingTimer(IN := FALSE);
            PressureTimer(IN := FALSE);
            HoldTimer(IN := FALSE);
            CoolingTimer(IN := FALSE);
        END_IF;
        
    1: (* Charging - Add ammonia and CO2 *)
        (* Add ammonia *)
        AmmoniaAddFB(TargetWeight := TargetAmmoniaWeight, 
                    CurrentWeight := CurrentWeight, 
                    ValveFeedback := ValveFeedbackAmmonia, 
                    Tolerance := WeightTolerance);
        AmmoniaValveCommand := AmmoniaAddFB.ValveOn;
        
        (* Add CO2 after ammonia *)
        IF AmmoniaAddFB.AdditionDone THEN
            CO2AddFB(TargetWeight := TargetCO2Weight, 
                    CurrentWeight := CurrentWeight, 
                    ValveFeedback := ValveFeedbackCO2, 
                    Tolerance := WeightTolerance);
            CO2ValveCommand := CO2AddFB.ValveOn;
        END_IF;
        
        ChargingTimer(IN := TRUE, PT := CHARGING_TIMEOUT);
        
        IF AmmoniaAddFB.Fault OR CO2AddFB.Fault OR ChargingTimer.Q THEN
            FaultDetected := TRUE;
            ReactionPhase := 0;
        ELSIF CO2AddFB.AdditionDone THEN
            ChargingTimer(IN := FALSE);
            ReactionPhase := 2; (* Transition to heating *)
        END_IF;
        
    2: (* Heating - Reach 180°C *)
        HeatingFB(TargetTemp := TargetReactionTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        MixingFB(TargetSpeed := MixingSpeed, 
                Feedback := AgitatorFeedback);
        HeaterCommand := HeatingFB.HeaterOn;
        AgitatorCommand := MixingFB.MixerOn;
        HeatingTimer(IN := TRUE, PT := HEATING_TIMEOUT);
        
        (* Interlock: Mixing must be stable *)
        IF HeatingFB.Fault OR MixingFB.Fault OR HeatingTimer.Q THEN
            FaultDetected := TRUE;
            ReactionPhase := 0;
        ELSIF HeatingFB.HeatingDone AND MixingFB.MixingStable THEN
            HeatingTimer(IN := FALSE);
            ReactionPhase := 3; (* Transition to pressure regulation *)
        END_IF;
        
    3: (* Pressure Regulation - Maintain 140 bar *)
        PressureFB(TargetPressure := TargetReactionPressure, 
                  CurrentPressure := CurrentPressure, 
                  Tolerance := PressureTolerance);
        MixingFB(TargetSpeed := MixingSpeed, 
                Feedback := AgitatorFeedback);
        PressureValveCommand := PressureFB.ValveOn;
        AgitatorCommand := MixingFB.MixerOn;
        PressureTimer(IN := TRUE, PT := PRESSURE_TIMEOUT);
        
        IF PressureFB.Fault OR MixingFB.Fault OR PressureTimer.Q THEN
            FaultDetected := TRUE;
            ReactionPhase := 0;
        ELSIF PressureFB.PressureStable AND MixingFB.MixingStable THEN
            PressureTimer(IN := FALSE);
            ReactionPhase := 4; (* Transition to hold *)
        END_IF;
        
    4: (* Reaction Hold - Maintain conditions for 30 minutes *)
        HeatingFB(TargetTemp := TargetReactionTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        PressureFB(TargetPressure := TargetReactionPressure, 
                  CurrentPressure := CurrentPressure, 
                  Tolerance := PressureTolerance);
        MixingFB(TargetSpeed := MixingSpeed, 
                Feedback := AgitatorFeedback);
        HeaterCommand := HeatingFB.HeaterOn;
        PressureValveCommand := PressureFB.ValveOn;
        AgitatorCommand := MixingFB.MixerOn;
        HoldTimer(IN := TRUE, PT := ReactionHoldTime);
        
        IF HeatingFB.Fault OR PressureFB.Fault OR MixingFB.Fault THEN
            FaultDetected := TRUE;
            ReactionPhase := 0;
        ELSIF HoldTimer.Q THEN
            HoldTimer(IN := FALSE);
            ReactionPhase := 5; (* Transition to cooling *)
        END_IF;
        
    5: (* Cooling - Cool to 80°C *)
        CoolingFB(TargetTemp := TargetCoolingTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        MixingFB(TargetSpeed := MixingSpeed, 
                Feedback := AgitatorFeedback);
        CoolerCommand := CoolingFB.CoolerOn;
        AgitatorCommand := MixingFB.MixerOn;
        CoolingTimer(IN := TRUE, PT := COOLING_TIMEOUT);
        
        (* Release pressure during cooling *)
        PressureValveCommand := FALSE;
        
        IF CoolingFB.Fault OR MixingFB.Fault OR CoolingTimer.Q THEN
            FaultDetected := TRUE;
            ReactionPhase := 0;
        ELSIF CoolingFB.CoolingDone THEN
            CoolingTimer(IN := FALSE);
            ReactionComplete := TRUE;
            ReactionPhase := 6; (* Transition to complete *)
        END_IF;
        
    6: (* Complete - Reaction finished *)
        HeaterCommand := FALSE;
        CoolerCommand := FALSE;
        AgitatorCommand := FALSE;
        AmmoniaValveCommand := FALSE;
        CO2ValveCommand := FALSE;
        PressureValveCommand := FALSE;
        (* Await transfer or next recipe step *)
END_CASE;

(* Fault handling *)
IF FaultDetected THEN
    HeaterCommand := FALSE;
    CoolerCommand := FALSE;
    AgitatorCommand := FALSE;
    AmmoniaValveCommand := FALSE;
    CO2ValveCommand := FALSE;
    PressureValveCommand := FALSE;
    (* Fault must be acknowledged by higher-level control *)
END_IF;

END_PROGRAM
