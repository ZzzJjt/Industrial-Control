(* IEC 61131-3 Structured Text program for ISA-88-compliant PVC batch production *)
(* Controls Polymerize, Decover, and Dry phases with modular operations *)

PROGRAM PVCPolymerizationControl
VAR
    (* Batch control variables *)
    BatchState : INT := 0; (* 0=Idle, 1=Polymerize, 2=Decover, 3=Dry, 4=Complete *)
    PolymerizePhase : INT := 0; (* Polymerize sub-states: 0=Idle, 1=Evacuate, 2=ChargeWater, 3=ChargeVCM, 4=React *)
    StartBatch : BOOL := FALSE; (* Command to start batch *)
    BatchComplete : BOOL := FALSE; (* Flag for batch completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for process issues *)
    
    (* Process parameters *)
    TargetVacuumPressure : REAL := 50.0; (* Target reactor vacuum in mbar *)
    VacuumTolerance : REAL := 5.0; (* Acceptable vacuum deviation *)
    TargetWaterWeight : REAL := 5000.0; (* Demineralized water weight in kg *)
    TargetSurfactantWeight : REAL := 50.0; (* Surfactant weight in kg *)
    TargetVCMWeight : REAL := 3000.0; (* VCM weight in kg *)
    TargetCatalystWeight : REAL := 5.0; (* Catalyst weight in kg *)
    WeightTolerance : REAL := 10.0; (* Acceptable weight deviation in kg *)
    TargetReactionTemp : REAL := 57.5; (* Target reaction temperature in Celsius *)
    TempTolerance : REAL := 2.5; (* Acceptable temperature deviation *)
    TargetReactionPressure : REAL := 9.0; (* Target reaction pressure in bar *)
    PressureDropThreshold : REAL := 2.0; (* Pressure drop indicating reaction completion in bar *)
    MixingSpeed : REAL := 200.0; (* Agitator speed in RPM *)
    TargetDryingTemp : REAL := 80.0; (* Target drying temperature *)
    DryingVacuumPressure : REAL := 100.0; (* Target drying vacuum in mbar *)
    DryingTime : TIME := T#3h; (* Drying duration *)
    
    (* Input variables *)
    CurrentPressure : REAL; (* Current reactor/dryer pressure *)
    CurrentTemp : REAL; (* Current reactor/dryer temperature *)
    CurrentWeight : REAL; (* Current reactor weight from load cells *)
    AgitatorFeedback : BOOL; (* Agitator operational status *)
    VacuumPumpFeedback : BOOL; (* Vacuum pump status *)
    ValveFeedbackWater : BOOL; (* Water valve status *)
    ValveFeedbackSurfactant : BOOL; (* Surfactant valve status *)
    ValveFeedbackVCM : BOOL; (* VCM valve status *)
    ValveFeedbackCatalyst : BOOL; (* Catalyst valve status *)
    DepressurizationValveFeedback : BOOL; (* Depressurization valve status *)
    
    (* Output variables *)
    VacuumPumpCommand : BOOL; (* Command to vacuum pump *)
    WaterValveCommand : BOOL; (* Command to water valve *)
    SurfactantValveCommand : BOOL; (* Command to surfactant valve *)
    VCMValveCommand : BOOL; (* Command to VCM valve *)
    CatalystValveCommand : BOOL; (* Command to catalyst valve *)
    HeaterCommand : BOOL; (* Command to heater *)
    AgitatorCommand : BOOL; (* Command to agitator *)
    DepressurizationValveCommand : BOOL; (* Command to depressurization valve *)
    
    (* Timers *)
    EvacuationTimer : TON; (* Timer for reactor evacuation *)
    ChargeTimer : TON; (* Timer for material charging *)
    ReactionTimer : TON; (* Timer for reaction monitoring *)
    DepressurizationTimer : TON; (* Timer for depressurization *)
    DryingTimer : TON; (* Timer for drying *)
    
    (* Timer presets *)
    EVACUATION_TIMEOUT : TIME := T#5m; (* Max time to reach vacuum *)
    CHARGE_TIMEOUT : TIME := T#10m; (* Max time to charge materials *)
    REACTION_TIMEOUT : TIME := T#6h; (* Max reaction time *)
    DEPRESSURIZATION_TIMEOUT : TIME := T#5m; (* Max depressurization time *)
    DRYING_TIMEOUT : TIME := T#30m; (* Max time to reach drying conditions *)
END_VAR

(* Modular function block declarations *)
FUNCTION_BLOCK EvacuateReactor
    VAR_INPUT
        TargetPressure : REAL;
        CurrentPressure : REAL;
        PumpFeedback : BOOL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        PumpOn : BOOL;
        EvacuationDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control vacuum pump to remove oxygen *)
    IF CurrentPressure <= TargetPressure + Tolerance THEN
        EvacuationDone := TRUE;
        PumpOn := FALSE;
    ELSIF PumpFeedback THEN
        PumpOn := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK AddDemineralizedWater
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
    (* Control water valve and verify weight *)
    IF ABS(CurrentWeight - TargetWeight) <= Tolerance THEN
        AdditionDone := TRUE;
        ValveOn := FALSE;
    ELSIF ValveFeedback THEN
        ValveOn := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

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
    (* Generic material addition for surfactants, VCM, catalyst *)
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

FUNCTION_BLOCK DepressurizeReactor
    VAR_INPUT
        CurrentPressure : REAL;
        ValveFeedback : BOOL;
    END_VAR
    VAR_OUTPUT
        ValveOn : BOOL;
        DepressurizationDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control depressurization valve *)
    IF CurrentPressure <= 1.0 THEN (* Atmospheric pressure *)
        DepressurizationDone := TRUE;
        ValveOn := FALSE;
    ELSIF ValveFeedback THEN
        ValveOn := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    EvacuateFB : EvacuateReactor;
    WaterAddFB : AddDemineralizedWater;
    MaterialAddFB : AddMaterial;
    HeatingFB : StartHeating;
    MixingFB : StartMixing;
    DepressurizeFB : DepressurizeReactor;
END_VAR

(* Main batch control logic *)
CASE BatchState OF
    0: (* Idle - Waiting for start command *)
        IF StartBatch THEN
            BatchState := 1; (* Start polymerize phase *)
            PolymerizePhase := 0; (* Reset polymerize sub-state *)
            StartBatch := FALSE;
            (* Reset all timers *)
            EvacuationTimer(IN := FALSE);
            ChargeTimer(IN := FALSE);
            ReactionTimer(IN := FALSE);
            DepressurizationTimer(IN := FALSE);
            DryingTimer(IN := FALSE);
        END_IF;
        
    1: (* Polymerize phase *)
        CASE PolymerizePhase OF
            0: (* Polymerize Idle - Start evacuation *)
                PolymerizePhase := 1;
                
            1: (* Evacuate reactor *)
                EvacuateFB(TargetPressure := TargetVacuumPressure, 
                          CurrentPressure := CurrentPressure, 
                          PumpFeedback := VacuumPumpFeedback, 
                          Tolerance := VacuumTolerance);
                VacuumPumpCommand := EvacuateFB.PumpOn;
                EvacuationTimer(IN := TRUE, PT := EVACUATION_TIMEOUT);
                
                IF EvacuateFB.Fault OR EvacuationTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF EvacuateFB.EvacuationDone THEN
                    EvacuationTimer(IN := FALSE);
                    PolymerizePhase := 2; (* Transition to water charge *)
                END_IF;
                
            2: (* Charge water and surfactants *)
                (* Add demineralized water *)
                WaterAddFB(TargetWeight := TargetWaterWeight, 
                          CurrentWeight := CurrentWeight, 
                          ValveFeedback := ValveFeedbackWater, 
                          Tolerance := WeightTolerance);
                WaterValveCommand := WaterAddFB.ValveOn;
                
                (* Add surfactants after water *)
                IF WaterAddFB.AdditionDone THEN
                    MaterialAddFB(TargetWeight := TargetSurfactantWeight, 
                                 CurrentWeight := CurrentWeight, 
                                 ValveFeedback := ValveFeedbackSurfactant, 
                                 Tolerance := WeightTolerance);
                    SurfactantValveCommand := MaterialAddFB.ValveOn;
                END_IF;
                
                ChargeTimer(IN := TRUE, PT := CHARGE_TIMEOUT);
                
                IF WaterAddFB.Fault OR MaterialAddFB.Fault OR ChargeTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF MaterialAddFB.AdditionDone THEN
                    ChargeTimer(IN := FALSE);
                    PolymerizePhase := 3; (* Transition to VCM charge *)
                END_IF;
                
            3: (* Charge VCM and catalyst *)
                (* Add VCM *)
                MaterialAddFB(TargetWeight := TargetVCMWeight, 
                             CurrentWeight := CurrentWeight, 
                             ValveFeedback := ValveFeedbackVCM, 
                             Tolerance := WeightTolerance);
                VCMValveCommand := MaterialAddFB.ValveOn;
                
                (* Add catalyst after VCM *)
                IF MaterialAddFB.AdditionDone THEN
                    MaterialAddFB(TargetWeight := TargetCatalystWeight, 
                                 CurrentWeight := CurrentWeight, 
                                 ValveFeedback := ValveFeedbackCatalyst, 
                                 Tolerance := WeightTolerance);
                    CatalystValveCommand := MaterialAddFB.ValveOn;
                END_IF;
                
                ChargeTimer(IN := TRUE, PT := CHARGE_TIMEOUT);
                
                IF MaterialAddFB.Fault OR ChargeTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF MaterialAddFB.AdditionDone THEN
                    ChargeTimer(IN := FALSE);
                    PolymerizePhase := 4; (* Transition to reaction *)
                END_IF;
                
            4: (* Reaction sub-step *)
                (* Control temperature and mixing *)
                HeatingFB(TargetTemp := TargetReactionTemp, 
                         CurrentTemp := CurrentTemp, 
                         Tolerance := TempTolerance);
                MixingFB(TargetSpeed := MixingSpeed, 
                        Feedback := AgitatorFeedback);
                HeaterCommand := HeatingFB.HeaterOn;
                AgitatorCommand := MixingFB.MixerOn;
                ReactionTimer(IN := TRUE, PT := REACTION_TIMEOUT);
                
                (* Monitor pressure drop *)
                IF CurrentPressure <= TargetReactionPressure - PressureDropThreshold THEN
                    ReactionTimer(IN := FALSE);
                    BatchState := 2; (* Transition to decover *)
                ELSIF HeatingFB.Fault OR MixingFB.Fault OR ReactionTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                END_IF;
        END_CASE;
        
    2: (* Decover phase *)
        (* Stop heating and agitation *)
        HeaterCommand := FALSE;
        AgitatorCommand := FALSE;
        
        (* Depressurize reactor *)
        DepressurizeFB(CurrentPressure := CurrentPressure, 
                      ValveFeedback := DepressurizationValveFeedback);
        DepressurizationValveCommand := DepressurizeFB.ValveOn;
        DepressurizationTimer(IN := TRUE, PT := DEPRESSURIZATION_TIMEOUT);
        
        IF DepressurizeFB.Fault OR DepressurizationTimer.Q THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF DepressurizeFB.DepressurizationDone THEN
            DepressurizationTimer(IN := FALSE);
            BatchState := 3; (* Transition to drying *)
        END_IF;
        
    3: (* Dry phase *)
        (* Control drying temperature and vacuum *)
        HeatingFB(TargetTemp := TargetDryingTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        EvacuateFB(TargetPressure := DryingVacuumPressure, 
                  CurrentPressure := CurrentPressure, 
                  PumpFeedback := VacuumPumpFeedback, 
                  Tolerance := VacuumTolerance);
        HeaterCommand := HeatingFB.HeaterOn;
        VacuumPumpCommand := EvacuateFB.PumpOn;
        DryingTimer(IN := TRUE, PT := DRYING_TIMEOUT);
        
        (* Interlock: Vacuum must be stable before drying *)
        IF HeatingFB.Fault OR EvacuateFB.Fault OR DryingTimer.Q THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF HeatingFB.HeatingDone AND EvacuateFB.EvacuationDone THEN
            DryingTimer(IN := TRUE, PT := DryingTime);
            IF DryingTimer.Q THEN
                DryingTimer(IN := FALSE);
                BatchComplete := TRUE;
                BatchState := 4; (* Transition to complete *)
            END_IF;
        END_IF;
        
    4: (* Complete - Batch finished *)
        HeaterCommand := FALSE;
        VacuumPumpCommand := FALSE;
        DepressurizationValveCommand := FALSE;
        (* Await reset or transfer to next process *)
END_CASE;

(* Fault handling *)
IF FaultDetected THEN
    HeaterCommand := FALSE;
    AgitatorCommand := FALSE;
    VacuumPumpCommand := FALSE;
    WaterValveCommand := FALSE;
    SurfactantValveCommand := FALSE;
    VCMValveCommand := FALSE;
    CatalystValveCommand := FALSE;
    DepressurizationValveCommand := FALSE;
    (* Fault must be acknowledged by higher-level control *)
END_IF;

END_PROGRAM
