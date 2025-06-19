(* IEC 61131-3 Structured Text program for ISA-88-compliant polyethylene batch production *)
(* Controls all phases: Raw Material Preparation, Polymerization, Quenching, Drying, Pelletizing, Quality Control, Packaging *)
(* Follows ISA-88 procedural and equipment control separation with modular function blocks *)

PROGRAM PolyethyleneBatchControl
VAR
    (* Batch control variables *)
    BatchState : INT := 0; (* 0=Idle, 1=RawPrep, 2=Polymerize, 3=Quench, 4=Dry, 5=Pelletize, 6=QualityControl, 7=Package, 8=Complete *)
    SubPhase : INT := 0; (* Sub-state for multi-step phases *)
    StartBatch : BOOL := FALSE; (* Command to start batch *)
    BatchComplete : BOOL := FALSE; (* Flag for batch completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for process issues *)
    BatchRejected : BOOL := FALSE; (* Flag for quality control failure *)
    
    (* Process parameters *)
    TargetSolventWeight : REAL := 5000.0; (* Solvent weight in kg *)
    TargetEthyleneWeight : REAL := 3000.0; (* Ethylene weight in kg *)
    TargetCatalystWeight : REAL := 5.0; (* Catalyst weight in kg *)
    WeightTolerance : REAL := 10.0; (* Acceptable weight deviation in kg *)
    PrepMixingSpeed : REAL := 100.0; (* Mixing speed in preparation in RPM *)
    PrepTemp : REAL := 30.0; (* Preparation temperature in Celsius *)
    PrepMixingTime : TIME := T#10m; (* Preparation mixing duration *)
    
    TargetReactionTemp : REAL := 80.0; (* Polymerization temperature *)
    TempTolerance : REAL := 2.0; (* Acceptable temperature deviation *)
    TargetReactionPressure : REAL := 15.0; (* Polymerization pressure in bar *)
    PressureTolerance : REAL := 0.5; (* Acceptable pressure deviation *)
    ReactionMixingSpeed : REAL := 200.0; (* Reaction mixing speed in RPM *)
    ReactionViscosityThreshold : REAL := 1000.0; (* Viscosity indicating reaction completion in cP *)
    ReactionTime : TIME := T#4h; (* Max polymerization duration *)
    
    TargetQuenchWeight : REAL := 1000.0; (* Quenching water weight in kg *)
    QuenchTemp : REAL := 20.0; (* Quenching temperature *)
    QuenchMixingSpeed : REAL := 50.0; (* Quenching mixing speed in RPM *)
    QuenchTime : TIME := T#30m; (* Quenching duration *)
    
    TargetDryingTemp : REAL := 90.0; (* Drying temperature *)
    DryingVacuumPressure : REAL := 100.0; (* Drying vacuum pressure in mbar *)
    DryingTime : TIME := T#2h; (* Drying duration *)
    
    ExtrusionTemp : REAL := 200.0; (* Pelletizing extrusion temperature *)
    PelletSize : REAL := 2.5; (* Target pellet size in mm *)
    PelletSizeTolerance : REAL := 0.5; (* Acceptable pellet size deviation *)
    
    TargetDensity : REAL := 0.95; (* Target pellet density in g/cm³ *)
    DensityTolerance : REAL := 0.01; (* Acceptable density deviation *)
    TargetMeltIndex : REAL := 1.5; (* Target melt index in g/10 min *)
    MeltIndexTolerance : REAL := 0.5; (* Acceptable melt index deviation *)
    
    BagWeight : REAL := 25.0; (* Packaging bag weight in kg *)
    
    (* Input variables *)
    CurrentWeight : REAL; (* Current vessel weight from load cells *)
    CurrentTemp : REAL; (* Current temperature *)
    CurrentPressure : REAL; (* Current pressure *)
    CurrentViscosity : REAL; (* Current slurry viscosity *)
    MixerFeedback : BOOL; (* Mixer operational status *)
    ValveFeedbackSolvent : BOOL; (* Solvent valve status *)
    ValveFeedbackEthylene : BOOL; (* Ethylene valve status *)
    ValveFeedbackCatalyst : BOOL; (* Catalyst valve status *)
    ValveFeedbackWater : BOOL; (* Quenching water valve status *)
    VacuumPumpFeedback : BOOL; (* Vacuum pump status *)
    ExtruderFeedback : BOOL; (* Extruder operational status *)
    PelletSizeActual : REAL; (* Measured pellet size *)
    DensityActual : REAL; (* Measured pellet density *)
    MeltIndexActual : REAL; (* Measured melt index *)
    PackagingFeedback : BOOL; (* Packaging system status *)
    
    (* Output variables *)
    SolventValveCommand : BOOL; (* Command to solvent valve *)
    EthyleneValveCommand : BOOL; (* Command to ethylene valve *)
    CatalystValveCommand : BOOL; (* Command to catalyst valve *)
    WaterValveCommand : BOOL; (* Command to quenching water valve *)
    HeaterCommand : BOOL; (* Command to heater *)
    MixerCommand : BOOL; (* Command to mixer *)
    PressureValveCommand : BOOL; (* Command to pressure valve *)
    VacuumPumpCommand : BOOL; (* Command to vacuum pump *)
    ExtruderCommand : BOOL; (* Command to extruder *)
    PelletCutterCommand : BOOL; (* Command to pellet cutter *)
    PackagingCommand : BOOL; (* Command to packaging system *)
    
    (* Timers *)
    PrepMixingTimer : TON; (* Timer for preparation mixing *)
    ReactionTimer : TON; (* Timer for polymerization *)
    QuenchTimer : TON; (* Timer for quenching *)
    DryingTimer : TON; (* Timer for drying *)
    PelletizingTimer : TON; (* Timer for pelletizing *)
    QualityControlTimer : TON; (* Timer for quality testing *)
    PackagingTimer : TON; (* Timer for packaging *)
    
    (* Timer presets *)
    PREP_TIMEOUT : TIME := T#15m; (* Max preparation time *)
    REACTION_TIMEOUT : TIME := T#5h; (* Max polymerization time *)
    QUENCH_TIMEOUT : TIME := T#40m; (* Max quenching time *)
    DRYING_TIMEOUT : TIME := T#2h30m; (* Max drying time *)
    PELLETIZING_TIMEOUT : TIME := T#30m; (* Max pelletizing time *)
    QC_TIMEOUT : TIME := T#10m; (* Max quality control time *)
    PACKAGING_TIMEOUT : TIME := T#20m; (* Max packaging time *)
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
    (* Control mixer and verify operation *)
    IF Feedback THEN
        MixerOn := TRUE;
        MixingStable := TRUE;
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

FUNCTION_BLOCK ControlPressure
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

FUNCTION_BLOCK StartVacuum
    VAR_INPUT
        TargetPressure : REAL;
        CurrentPressure : REAL;
        PumpFeedback : BOOL;
        Tolerance : REAL;
    END_VAR
    VAR_OUTPUT
        PumpOn : BOOL;
        VacuumStable : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control vacuum pump *)
    IF CurrentPressure <= TargetPressure + Tolerance THEN
        VacuumStable := TRUE;
        PumpOn := FALSE;
    ELSIF PumpFeedback THEN
        PumpOn := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK BeginPelletizing
    VAR_INPUT
        TargetTemp : REAL;
        CurrentTemp : REAL;
        TargetSize : REAL;
        ActualSize : REAL;
        ExtruderFeedback : BOOL;
        TempTolerance : REAL;
        SizeTolerance : REAL;
    END_VAR
    VAR_OUTPUT
        ExtruderOn : BOOL;
        CutterOn : BOOL;
        PelletizingDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control extruder and pellet cutter *)
    IF ABS(CurrentTemp - TargetTemp) <= TempTolerance AND 
       ABS(ActualSize - TargetSize) <= SizeTolerance AND 
       ExtruderFeedback THEN
        ExtruderOn := TRUE;
        CutterOn := TRUE;
        PelletizingDone := TRUE;
    ELSIF ExtruderFeedback THEN
        ExtruderOn := TRUE;
        CutterOn := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK PerformQualityControl
    VAR_INPUT
        TargetDensity : REAL;
        ActualDensity : REAL;
        TargetMeltIndex : REAL;
        ActualMeltIndex : REAL;
        DensityTolerance : REAL;
        MeltIndexTolerance : REAL;
    END_VAR
    VAR_OUTPUT
        QualityPassed : BOOL;
        Fault : BOOL;
    END_VAR
    (* Verify pellet quality *)
    IF ABS(ActualDensity - TargetDensity) <= DensityTolerance AND 
       ABS(ActualMeltIndex - TargetMeltIndex) <= MeltIndexTolerance THEN
        QualityPassed := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartPackaging
    VAR_INPUT
        TargetBagWeight : REAL;
        Feedback : BOOL;
    END_VAR
    VAR_OUTPUT
        PackagingOn : BOOL;
        PackagingDone : BOOL;
        Fault : BOOL;
    END_VAR
    (* Control packaging system *)
    IF Feedback THEN
        PackagingOn := TRUE;
        PackagingDone := TRUE;
    ELSE
        Fault := TRUE;
    END_IF;
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    SolventAddFB : AddMaterial;
    EthyleneAddFB : AddMaterial;
    CatalystAddFB : AddMaterial;
    WaterAddFB : AddMaterial;
    MixingFB : StartMixing;
    HeatingFB : StartHeating;
    PressureFB : ControlPressure;
    VacuumFB : StartVacuum;
    PelletizingFB : BeginPelletizing;
    QualityControlFB : PerformQualityControl;
    PackagingFB : StartPackaging;
END_VAR

(* Main batch control logic *)
CASE BatchState OF
    0: (* Idle - Waiting for start command *)
        IF StartBatch THEN
            BatchState := 1; (* Start raw material preparation *)
            SubPhase := 0; (* Reset sub-phase *)
            StartBatch := FALSE;
            (* Reset all timers *)
            PrepMixingTimer(IN := FALSE);
            ReactionTimer(IN := FALSE);
            QuenchTimer(IN := FALSE);
            DryingTimer(IN := FALSE);
            PelletizingTimer(IN := FALSE);
            QualityControlTimer(IN := FALSE);
            PackagingTimer(IN := FALSE);
        END_IF;
        
    1: (* Raw Material Preparation *)
        CASE SubPhase OF
            0: (* Start solvent addition *)
                SubPhase := 1;
                
            1: (* Add solvent *)
                SolventAddFB(TargetWeight := TargetSolventWeight, 
                            CurrentWeight := CurrentWeight, 
                            ValveFeedback := ValveFeedbackSolvent, 
                            Tolerance := WeightTolerance);
                SolventValveCommand := SolventAddFB.ValveOn;
                PrepMixingTimer(IN := TRUE, PT := PREP_TIMEOUT);
                
                IF SolventAddFB.Fault OR PrepMixingTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF SolventAddFB.AdditionDone THEN
                    PrepMixingTimer(IN := FALSE);
                    SubPhase := 2; (* Transition to ethylene *)
                END_IF;
                
            2: (* Add ethylene *)
                EthyleneAddFB(TargetWeight := TargetEthyleneWeight, 
                             CurrentWeight := CurrentWeight, 
                             ValveFeedback := ValveFeedbackEthylene, 
                             Tolerance := WeightTolerance);
                EthyleneValveCommand := EthyleneAddFB.ValveOn;
                PrepMixingTimer(IN := TRUE, PT := PREP_TIMEOUT);
                
                IF EthyleneAddFB.Fault OR PrepMixingTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF EthyleneAddFB.AdditionDone THEN
                    PrepMixingTimer(IN := FALSE);
                    SubPhase := 3; (* Transition to catalyst *)
                END_IF;
                
            3: (* Add catalyst *)
                CatalystAddFB(TargetWeight := TargetCatalystWeight, 
                             CurrentWeight := CurrentWeight, 
                             ValveFeedback := ValveFeedbackCatalyst, 
                             Tolerance := WeightTolerance);
                CatalystValveCommand := CatalystAddFB.ValveOn;
                PrepMixingTimer(IN := TRUE, PT := PREP_TIMEOUT);
                
                IF CatalystAddFB.Fault OR PrepMixingTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF CatalystAddFB.AdditionDone THEN
                    PrepMixingTimer(IN := FALSE);
                    SubPhase := 4; (* Transition to mixing *)
                END_IF;
                
            4: (* Mix materials *)
                HeatingFB(TargetTemp := PrepTemp, 
                         CurrentTemp := CurrentTemp, 
                         Tolerance := TempTolerance);
                MixingFB(TargetSpeed := PrepMixingSpeed, 
                        Feedback := MixerFeedback);
                HeaterCommand := HeatingFB.HeaterOn;
                MixerCommand := MixingFB.MixerOn;
                PrepMixingTimer(IN := TRUE, PT := PrepMixingTime);
                
                IF HeatingFB.Fault OR MixingFB.Fault THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF HeatingFB.HeatingDone AND MixingFB.MixingStable AND PrepMixingTimer.Q THEN
                    PrepMixingTimer(IN := FALSE);
                    BatchState := 2; (* Transition to polymerization *)
                END_IF;
        END_CASE;
        
    2: (* Polymerization *)
        (* Control temperature, pressure, and mixing *)
        HeatingFB(TargetTemp := TargetReactionTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        PressureFB(TargetPressure := TargetReactionPressure, 
                  CurrentPressure := CurrentPressure, 
                  Tolerance := PressureTolerance);
        MixingFB(TargetSpeed := ReactionMixingSpeed, 
                Feedback := MixerFeedback);
        HeaterCommand := HeatingFB.HeaterOn;
        PressureValveCommand := PressureFB.ValveOn;
        MixerCommand := MixingFB.MixerOn;
        ReactionTimer(IN := TRUE, PT := REACTION_TIMEOUT);
        
        (* Check viscosity for reaction completion *)
        IF CurrentViscosity >= ReactionViscosityThreshold THEN
            ReactionTimer(IN := FALSE);
            BatchState := 3; (* Transition to quenching *)
        ELSIF HeatingFB.Fault OR PressureFB.Fault OR MixingFB.Fault OR ReactionTimer.Q THEN
            FaultDetected := TRUE;
            BatchState := 0;
        END_IF;
        
    3: (* Quenching *)
        (* Add water and mix *)
        WaterAddFB(TargetWeight := TargetQuenchWeight, 
                  CurrentWeight := CurrentWeight, 
                  ValveFeedback := ValveFeedbackWater, 
                  Tolerance := WeightTolerance);
        HeatingFB(TargetTemp := QuenchTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        MixingFB(TargetSpeed := QuenchMixingSpeed, 
                Feedback := MixerFeedback);
        WaterValveCommand := WaterAddFB.ValveOn;
        HeaterCommand := HeatingFB.HeaterOn;
        MixerCommand := MixingFB.MixerOn;
        QuenchTimer(IN := TRUE, PT := QuenchTime);
        
        IF WaterAddFB.Fault OR HeatingFB.Fault OR MixingFB.Fault THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF WaterAddFB.AdditionDone AND HeatingFB.HeatingDone AND MixingFB.MixingStable AND QuenchTimer.Q THEN
            QuenchTimer(IN := FALSE);
            BatchState := 4; (* Transition to drying *)
        END_IF;
        
    4: (* Drying *)
        (* Control temperature and vacuum *)
        HeatingFB(TargetTemp := TargetDryingTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        VacuumFB(TargetPressure := DryingVacuumPressure, 
                CurrentPressure := CurrentPressure, 
                PumpFeedback := VacuumPumpFeedback, 
                Tolerance := PressureTolerance);
        HeaterCommand := HeatingFB.HeaterOn;
        VacuumPumpCommand := VacuumFB.PumpOn;
        DryingTimer(IN := TRUE, PT := DryingTime);
        
        (* Interlock: Vacuum must be stable *)
        IF HeatingFB.Fault OR VacuumFB.Fault THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF HeatingFB.HeatingDone AND VacuumFB.VacuumStable AND DryingTimer.Q THEN
            DryingTimer(IN := FALSE);
            BatchState := 5; (* Transition to pelletizing *)
        END_IF;
        
    5: (* Pelletizing *)
        (* Control extrusion and cutting *)
        PelletizingFB(TargetTemp := ExtrusionTemp, 
                     CurrentTemp := CurrentTemp, 
                     TargetSize := PelletSize, 
                     ActualSize := PelletSizeActual, 
                     ExtruderFeedback := ExtruderFeedback, 
                     TempTolerance := TempTolerance, 
                     SizeTolerance := PelletSizeTolerance);
        ExtruderCommand := PelletizingFB.ExtruderOn;
        PelletCutterCommand := PelletizingFB.CutterOn;
        PelletizingTimer(IN := TRUE, PT := PELLETIZING_TIMEOUT);
        
        IF PelletizingFB.Fault OR PelletizingTimer.Q THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF PelletizingFB.PelletizingDone THEN
            PelletizingTimer(IN := FALSE);
            BatchState := 6; (* Transition to quality control *)
        END_IF;
        
    6: (* Quality Control *)
        (* Test pellet properties *)
        QualityControlFB(TargetDensity := TargetDensity, 
                        ActualDensity := DensityActual, 
                        TargetMeltIndex := TargetMeltIndex, 
                        ActualMeltIndex := MeltIndexActual, 
                        DensityTolerance := DensityTolerance, 
                        MeltIndexTolerance := MeltIndexTolerance);
        QualityControlTimer(IN := TRUE, PT := QC_TIMEOUT);
        
        IF QualityControlFB.Fault THEN
            BatchRejected := TRUE;
            BatchState := 0;
        ELSIF QualityControlFB.QualityPassed AND QualityControlTimer.Q THEN
            QualityControlTimer(IN := FALSE);
            BatchState := 7; (* Transition to packaging *)
        END_IF;
        
    7: (* Packaging and Storage *)
        (* Package pellets *)
        PackagingFB(TargetBagWeight := BagWeight, 
                   Feedback := PackagingFeedback);
        PackagingCommand := PackagingFB.PackagingOn;
        PackagingTimer(IN := TRUE, PT := PACKAGING_TIMEOUT);
        
        IF PackagingFB.Fault OR PackagingTimer.Q THEN
            FaultDetected := TRUE;
            BatchState := 0;
        ELSIF PackagingFB.PackagingDone THEN
            PackagingTimer(IN := FALSE);
            BatchComplete := TRUE;
            BatchState := 8; (* Transition to complete *)
        END_IF;
        
    8: (* Complete - Batch finished *)
        (* Reset all outputs *)
        SolventValveCommand := FALSE;
        EthyleneValveCommand := FALSE;
        CatalystValveCommand := FALSE;
        WaterValveCommand := FALSE;
        HeaterCommand := FALSE;
        MixerCommand := FALSE;
        PressureValveCommand := FALSE;
        VacuumPumpCommand := FALSE;
        ExtruderCommand := FALSE;
        PelletCutterCommand := FALSE;
        PackagingCommand := FALSE;
        (* Await reset or next batch *)
END_CASE;

(* Fault handling *)
IF FaultDetected OR BatchRejected THEN
    SolventValveCommand := FALSE;
    EthyleneValveCommand := FALSE;
    CatalystValveCommand := FALSE;
    WaterValveCommand := FALSE;
    HeaterCommand := FALSE;
    MixerCommand := FALSE;
    PressureValveCommand := FALSE;
    VacuumPumpCommand := FALSE;
    ExtruderCommand := FALSE;
    PelletCutterCommand := FALSE;
    PackagingCommand := FALSE;
    (* Fault or rejection must be acknowledged by higher-level control *)
END_IF;

END_PROGRAM
