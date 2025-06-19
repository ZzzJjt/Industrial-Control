(* IEC 61131-3 Structured Text program for ISA-88 Step B.2: Reaction *)
(* Implements reaction phase for adhesive production with modular operations *)

PROGRAM AdhesiveReactionStep
VAR
    (* Process parameters *)
    TargetReactionTemp : REAL := 85.0; (* Target reaction temperature in Celsius *)
    TempTolerance : REAL := 2.0; (* Acceptable temperature deviation *)
    MixingSpeed : REAL := 500.0; (* Mixer speed in RPM *)
    ReactionHoldTime : TIME := T#30m; (* Duration to maintain reaction conditions *)
    
    (* Phase control variables *)
    PhaseState : INT := 0; (* 0=Idle, 1=Heating, 2=Mixing, 3=Holding, 4=Complete *)
    ReactionComplete : BOOL := FALSE; (* Flag for step completion *)
    FaultDetected : BOOL := FALSE; (* Fault flag for temperature or equipment issues *)
    
    (* Input variables *)
    StartReaction : BOOL := FALSE; (* Command to start reaction step *)
    CurrentTemp : REAL; (* Current reactor temperature from sensor *)
    MixerFeedback : BOOL; (* Mixer operational status *)
    
    (* Timers *)
    HeatingTimer : TON; (* Timer for heating phase *)
    MixingTimer : TON; (* Timer for mixing stabilization *)
    HoldingTimer : TON; (* Timer for reaction hold duration *)
    
    (* Timer presets *)
    HEATING_TIMEOUT : TIME := T#5m; (* Max time to reach target temperature *)
    MIXING_STABILIZE_TIME : TIME := T#30s; (* Time to stabilize mixing *)
    
    (* Output variables *)
    HeaterCommand : BOOL; (* Command to heater *)
    MixerCommand : BOOL; (* Command to mixer *)
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

FUNCTION_BLOCK MaintainReaction
    VAR_INPUT
        CurrentTemp : REAL;
        TargetTemp : REAL;
        Tolerance : REAL;
        MixerStatus : BOOL;
    END_VAR
    VAR_OUTPUT
        HeaterOn : BOOL;
        MixerOn : BOOL;
        Fault : BOOL;
    END_VAR
    (* Logic to maintain temperature and mixing *)
END_FUNCTION_BLOCK

(* Instances of function blocks *)
VAR
    HeatingFB : StartHeating;
    MixingFB : StartMixing;
    ReactionFB : MaintainReaction;
END_VAR

(* Main reaction step logic *)
CASE PhaseState OF
    0: (* Idle - Waiting for start command *)
        IF StartReaction THEN
            PhaseState := 1; (* Transition to heating phase *)
            StartReaction := FALSE;
            HeatingTimer(IN := FALSE); (* Reset timers *)
            MixingTimer(IN := FALSE);
            HoldingTimer(IN := FALSE);
        END_IF;
        
    1: (* Heating phase *)
        (* Call heating function block *)
        HeatingFB(TargetTemp := TargetReactionTemp, 
                 CurrentTemp := CurrentTemp, 
                 Tolerance := TempTolerance);
        HeaterCommand := HeatingFB.HeaterOn;
        
        (* Start heating timer *)
        HeatingTimer(IN := TRUE, PT := HEATING_TIMEOUT);
        
        (* Check for completion or fault *)
        IF HeatingFB.Fault OR HeatingTimer.Q THEN
            FaultDetected := TRUE;
            PhaseState := 0; (* Return to idle on fault or timeout *)
        ELSIF HeatingFB.HeatingDone THEN
            HeatingTimer(IN := FALSE);
            PhaseState := 2; (* Transition to mixing phase *)
        END_IF;
        
    2: (* Mixing phase *)
        (* Call mixing function block *)
        MixingFB(TargetSpeed := MixingSpeed, 
                Feedback := MixerFeedback);
        MixerCommand := MixingFB.MixerOn;
        
        (* Start mixing stabilization timer *)
        MixingTimer(IN := TRUE, PT := MIXING_STABILIZE_TIME);
        
        (* Check for completion or fault *)
        IF MixingFB.Fault THEN
            FaultDetected := TRUE;
            PhaseState := 0;
        ELSIF MixingFB.MixingStable AND MixingTimer.Q THEN
            MixingTimer(IN := FALSE);
            PhaseState := 3; (* Transition to holding phase *)
        END_IF;
        
    3: (* Holding phase - Maintain reaction conditions *)
        (* Call reaction maintenance function block *)
        ReactionFB(CurrentTemp := CurrentTemp, 
                  TargetTemp := TargetReactionTemp, 
                  Tolerance := TempTolerance, 
                  MixerStatus := MixerFeedback);
        HeaterCommand := ReactionFB.HeaterOn;
        MixerCommand := ReactionFB.MixerOn;
        
        (* Start holding timer *)
        HoldingTimer(IN := TRUE, PT := ReactionHoldTime);
        
        (* Check for completion or fault *)
        IF ReactionFB.Fault THEN
            FaultDetected := TRUE;
            PhaseState := 0;
        ELSIF HoldingTimer.Q THEN
            ReactionComplete := TRUE;
            PhaseState := 4; (* Transition to complete *)
        END_IF;
        
    4: (* Complete - Step finished *)
        (* Ensure all equipment is off *)
        HeaterCommand := FALSE;
        MixerCommand := FALSE;
        (* Await reset or next recipe step from higher-level control *)
END_CASE;

(* Fault handling *)
IF FaultDetected THEN
    HeaterCommand := FALSE;
    MixerCommand := FALSE;
    (* Fault must be acknowledged by higher-level control *)
END_IF;

END_PROGRAM
