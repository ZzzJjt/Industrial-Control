(* IEC 61131-3 Structured Text program for ISA-88-compliant aspirin production *)
(* Implements Reaction, Crystallization, and Drying phases with modular control *)

PROGRAM AspirinBatchControl
VAR
    (* Batch control variables *)
    BatchState : INT := 0; (* 0=Idle, 1=Reaction, 2=Crystallization, 3=Drying, 4=Complete *)
    ReactionPhase : INT := 0; (* Reaction sub-states: 0=Idle, 1=Heating, 2=Mixing, 3=Holding *)
    StartBatch : BOOL := FALSE; (* Command to start batch *)
    BatchComplete : BOOL := FALSE; (* Flag for batch completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for process issues *)
    
    (* Reaction parameters *)
    TargetReactionTemp : REAL := 70.0; (* Target reaction temperature in Celsius *)
    TempTolerance : REAL := 2.0; (* Acceptable temperature deviation *)
    TargetPressure : REAL := 1.1; (* Target reactor pressure in bar *)
    PressureTolerance : REAL := 0.05; (* Acceptable pressure deviation *)
    MixingSpeedReaction : REAL := 300.0; (* Mixer speed in RPM for reaction *)
    ReactionHoldTime : TIME := T#2h; (* Duration to hold reaction conditions *)
    
    (* Crystallization parameters *)
    TargetCrystalTemp : REAL := 20.0; (* Target crystallization temperature *)
    MixingSpeedCrystal : REAL := 100.0; (* Mixer speed in RPM for crystallization *)
    CrystalHoldTime : TIME := T#1h; (* Duration for crystal formation *)
    
    (* Drying parameters *)
    TargetDryingTemp : REAL := 90.0; (* Target drying temperature *)
    DryingHoldTime : TIME := T#3h; (* Duration for drying *)
    VacuumPressure : REAL := 0.1; (* Target vacuum pressure in bar *)
    
    (* Input variables *)
    CurrentTemp : REAL; (* Current reactor/crystallizer/dryer temperature *)
    CurrentPressure : REAL; (* Current reactor/dryer pressure *)
    MixerFeedback : BOOL; (* Mixer operational status *)
    WaterValveFeedback : BOOL; (* Water valve status for crystallization *)
    VacuumPumpFeedback : BOOL; (* Vacuum pump status for drying *)
    
    (* Output variables *)
    HeaterCommand : BOOL; (* Command to heater *)
    MixerCommand : BOOL; (* Command to mixer *)
    PressureValveCommand : BOOL; (* Command to pressure control valve *)
    WaterValveCommand : BOOL; (* Command to water addition valve *)
    VacuumPumpCommand : BOOL; (* Command to vacuum pump *)
    
    (* Timers *)
    HeatingTimer : TON; (* Timer for heating phase *)
    MixingTimer : TON; (* Timer for mixing stabilization *)
    ReactionHoldTimer : TON; (* Timer for reaction hold *)
    CrystalCoolingTimer : TON; (* Timer for crystallization cooling *)
    CrystalHoldTimer : TON; (* Timer for crystallization hold *)
    DryingTimer : TON; (* Timer for drying *)
    
    (* Timer presets *)
    HEATING_TIMEOUT : TIME := T#10m; (* Max time to reach reaction temperature *)
    MIXING_STABILIZE_TIME : TIME := T#30s; (* Time to stabilize mixing *)
    COOLING_TIMEOUT : TIME := T#15m; (* Max time to reach crystallization temp *)
    DRYING_TIMEOUT : TIME := T#20m; (* Max time to reach drying temp *)
END_VAR

(* Modular function block declarations - assumed defined elsewhere *)
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
    (* Logic to control heater and verify temperature *)
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
    (* Logic to control mixer and verify operation *)
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
    (* Logic to control pressure valve *)
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
    (* Logic to control cooling system *)
END_FUNCTION_BLOCK

FUNCTION_BLOCK AddWater
    VAR_INPUT
        ValveFeedback : BOOL;
    END_VAR
    VAR_OUTPUT
        ValveOn : BOOL;
        WaterAdded : BOOL;
        Fault : BOOL;
    END_VAR
    (* Logic to control water addition *)
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartVacuum
    VAR_INPUT
        TargetPressure : REAL;
        CurrentPressure : REAL;
        PumpFeedback : BOOL;
    END_VAR
    VAR_OUTPUT
        PumpOn : BOOL;
        VacuumStable : BOOL;
        Fault : BOOL;
    END_VAR
    (* Logic to control vacuum pump *)
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    HeatingFB : StartHeating;
    MixingFB : StartMixing;
    PressureFB : ControlPressure;
    CoolingFB : StartCooling;
    WaterFB : AddWater;
    VacuumFB : StartVacuum;
END_VAR

(* Main batch control logic *)
CASE BatchState OF
    0: (* Idle - Waiting for start command *)
        IF StartBatch THEN
            BatchState := 1; (* Start reaction phase *)
            ReactionPhase := 0; (* Reset reaction sub-state *)
            StartBatch := FALSE;
            (* Reset all timers *)
            HeatingTimer(IN := FALSE);
            MixingTimer(IN := FALSE);
            ReactionHoldTimer(IN := FALSE);
            CrystalCoolingTimer(IN := FALSE);
            CrystalHoldTimer(IN := FALSE);
            DryingTimer(IN := FALSE);
        END_IF;
        
    1: (* Reaction phase *)
        CASE ReactionPhase OF
            0: (* Reaction Idle - Start heating *)
                ReactionPhase := 1;
                
            1: (* Heating sub-step *)
                HeatingFB(TargetTemp := TargetReactionTemp, 
                         CurrentTemp := CurrentTemp, 
                         Tolerance := TempTolerance);
                HeaterCommand := HeatingFB.HeaterOn;
                HeatingTimer(IN := TRUE, PT := HEATING_TIMEOUT);
                
                IF HeatingFB.Fault OR HeatingTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF HeatingFB.HeatingDone THEN
                    HeatingTimer(IN := FALSE);
                    ReactionPhase := 2; (* Transition to mixing *)
                END_IF;
                
            2: (* Mixing sub-step *)
                MixingFB(TargetSpeed := MixingSpeedReaction, 
                        Feedback := MixerFeedback);
                MixerCommand := MixingFB.MixerOn;
                MixingTimer(IN := TRUE, PT := MIXING_STABILIZE_TIME);
                
                (* Control pressure during mixing *)
                PressureFB(TargetPressure := TargetPressure, 
                          CurrentPressure := CurrentPressure, 
                          Tolerance := PressureTolerance);
                PressureValveCommand := PressureFB.ValveOn;
                
                IF MixingFB.Fault OR PressureFB.Fault THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF MixingFB.MixingStable AND MixingTimer.Q AND PressureFB.PressureStable THEN
                    MixingTimer(IN := FALSE);
                    ReactionPhase := 3; (* Transition to holding *)
                END_IF;
                
            3: (* Holding sub-step *)
                (* Maintain temperature, mixing, and pressure *)
                HeatingFB(TargetTemp := TargetReactionTemp, 
                         CurrentTemp := CurrentTemp, 
                         Tolerance := TempTolerance);
                MixingFB(TargetSpeed := MixingSpeedReaction, 
                        Feedback := MixerFeedback);
                PressureFB(TargetPressure := TargetPressure, 
                          CurrentPressure := CurrentPressure, 
                          Tolerance := PressureTolerance);
                HeaterCommand := HeatingFB.HeaterOn;
                MixerCommand := MixingFB.MixerOn;
                PressureValveCommand := PressureFB.ValveOn;
                
                ReactionHoldTimer(IN := TRUE, PT := ReactionHoldTime);
                
                IF HeatingFB.Fault OR MixingFB.Fault OR PressureFB.Fault THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF ReactionHoldTimer.Q THEN
                    ReactionHoldTimer(IN := FALSE);
                    BatchState := 2; (* Transition to crystallization *)
                END_IF;
        END_CASE;
        
    2: (* Crystallization phase *)
        CASE ReactionPhase OF
            0: (* Crystallization Idle - Start cooling *)
                HeaterCommand := FALSE; (* Ensure heater off *)
                MixerCommand := FALSE; (* Stop reactor mixing *)
                PressureValveCommand := FALSE; (* Release pressure *)
                ReactionPhase := 1;
                
            1: (* Cooling sub-step *)
                CoolingFB(TargetTemp := TargetCrystalTemp, 
                         CurrentTemp := CurrentTemp, 
                         Tolerance := TempTolerance);
                CrystalCoolingTimer(IN := TRUE, PT := COOLING_TIMEOUT);
                
                IF CoolingFB.Fault OR CrystalCoolingTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF CoolingFB.CoolingDone THEN
                    CrystalCoolingTimer(IN := FALSE);
                    ReactionPhase := 2; (* Transition to water addition *)
                END_IF;
                
            2: (* Water addition sub-step *)
                WaterFB(ValveFeedback := WaterValveFeedback);
                WaterValveCommand := WaterFB.ValveOn;
                
                IF WaterFB.Fault THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF WaterFB.WaterAdded THEN
                    ReactionPhase := 3; (* Transition to mixing *)
                END_IF;
                
            3: (* Mixing sub-step *)
                MixingFB(TargetSpeed := MixingSpeedCrystal, 
                        Feedback := MixerFeedback);
                MixerCommand := MixingFB.MixerOn;
                CrystalHoldTimer(IN := TRUE, PT := CrystalHoldTime);
                
                IF MixingFB.Fault THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF CrystalHoldTimer.Q THEN
                    CrystalHoldTimer(IN := FALSE);
                    BatchState := 3; (* Transition to drying *)
                END_IF;
        END_CASE;
        
    3: (* Drying phase *)
        CASE ReactionPhase OF
            0: (* Drying Idle - Start heating *)
                MixerCommand := FALSE; (* Ensure crystallizer mixer off *)
                WaterValveCommand := FALSE; (* Ensure water valve closed *)
                ReactionPhase := 1;
                
            1: (* Heating and vacuum sub-step *)
                HeatingFB(TargetTemp := TargetDryingTemp, 
                         CurrentTemp := CurrentTemp, 
                         Tolerance := TempTolerance);
                VacuumFB(TargetPressure := VacuumPressure, 
                        CurrentPressure := CurrentPressure, 
                        PumpFeedback := VacuumPumpFeedback);
                HeaterCommand := HeatingFB.HeaterOn;
                VacuumPumpCommand := VacuumFB.PumpOn;
                DryingTimer(IN := TRUE, PT := DRYING_TIMEOUT);
                
                (* Interlock: Vacuum must be stable before maintaining drying *)
                IF HeatingFB.Fault OR VacuumFB.Fault OR DryingTimer.Q THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF HeatingFB.HeatingDone AND VacuumFB.VacuumStable THEN
                    DryingTimer(IN := FALSE);
                    ReactionPhase := 2; (* Transition to drying hold *)
                END_IF;
                
            2: (* Drying hold sub-step *)
                HeatingFB(TargetTemp := TargetDryingTemp, 
                         CurrentTemp := CurrentTemp, 
                         Tolerance := TempTolerance);
                VacuumFB(TargetPressure := VacuumPressure, 
                        CurrentPressure := CurrentPressure, 
                        PumpFeedback := VacuumPumpFeedback);
                HeaterCommand := HeatingFB.HeaterOn;
                VacuumPumpCommand := VacuumFB.PumpOn;
                DryingTimer(IN := TRUE, PT := DryingHoldTime);
                
                IF HeatingFB.Fault OR VacuumFB.Fault THEN
                    FaultDetected := TRUE;
                    BatchState := 0;
                ELSIF DryingTimer.Q THEN
                    DryingTimer(IN := FALSE);
                    BatchComplete := TRUE;
                    BatchState := 4; (* Transition to complete *)
                END_IF;
        END_CASE;
        
    4: (* Complete - Batch finished *)
        HeaterCommand := FALSE;
        VacuumPumpCommand := FALSE;
        (* Await reset or next recipe step *)
END_CASE;

(* Fault handling *)
IF FaultDetected THEN
    HeaterCommand := FALSE;
    MixerCommand := FALSE;
    PressureValveCommand := FALSE;
    WaterValveCommand := FALSE;
    VacuumPumpCommand := FALSE;
    (* Fault must be acknowledged by higher-level control *)
END_IF;

END_PROGRAM
