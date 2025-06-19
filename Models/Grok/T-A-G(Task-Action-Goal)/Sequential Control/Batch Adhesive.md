(* ISA-88 Compliant Structured Text Program for Adhesive Production *)
(* Step B.2: Reaction Phase *)
(* Conforms to IEC 61131-3 Standard *)

PROGRAM AdhesiveReactionControl
VAR
    (* ISA-88 Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: Heating, 2: Mixing, 3: Holding, 4: Complete, 5: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters *)
    ReactionTempSetpoint: REAL := 120.0; (* Target reaction temperature in °C *)
    TempTolerance: REAL := 2.0; (* Acceptable temperature deviation *)
    MixingSpeedSetpoint: REAL := 500.0; (* Mixer speed in RPM *)
    SpeedTolerance: REAL := 50.0; (* Acceptable speed deviation *)
    ReactionHoldTime: TIME := T#30m; (* Duration for holding reaction *)
    MaxPhaseTime: TIME := T#45m; (* Maximum duration for reaction phase *)
    
    (* Process Inputs *)
    CurrentTemp: REAL; (* Current reactor temperature *)
    CurrentMixingSpeed: REAL; (* Current mixer speed *)
    
    (* Process Outputs *)
    HeaterOn: BOOL; (* Heater control *)
    MixerOn: BOOL; (* Mixer control *)
    ReactionComplete: BOOL; (* Phase completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Timer for overall phase duration *)
    HoldTimer: TON; (* Timer for holding phase *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
END_VAR

(* Modular Function Blocks *)
FUNCTION_BLOCK StartHeating
    VAR_INPUT
        TempSetpoint: REAL;
        CurrentTemp: REAL;
        Tolerance: REAL;
    END_VAR
    VAR_OUTPUT
        HeaterCmd: BOOL;
        HeatingComplete: BOOL;
    END_VAR
    (* Logic to start and maintain heating *)
    HeaterCmd := (CurrentTemp < (TempSetpoint - Tolerance));
    HeatingComplete := (ABS(CurrentTemp - TempSetpoint) <= Tolerance);
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartMixing
    VAR_INPUT
        SpeedSetpoint: REAL;
        CurrentSpeed: REAL;
        Tolerance: REAL;
    END_VAR
    VAR_OUTPUT
        MixerCmd: BOOL;
        MixingComplete: BOOL;
    END_VAR
    (* Logic to start and maintain mixing *)
    MixerCmd := (CurrentSpeed < (SpeedSetpoint - Tolerance));
    MixingComplete := (ABS(CurrentSpeed - SpeedSetpoint) <= Tolerance);
END_FUNCTION_BLOCK

FUNCTION_BLOCK HoldReaction
    VAR_INPUT
        HoldDuration: TIME;
        Timer: TON;
    END_VAR
    VAR_OUTPUT
        HoldComplete: BOOL;
    END_VAR
    (* Logic to hold reaction for specified duration *)
    Timer(IN := TRUE, PT := HoldDuration);
    HoldComplete := Timer.Q;
END_FUNCTION_BLOCK

(* Main Phase Control Logic *)
VAR
    HeatingFB: StartHeating;
    MixingFB: StartMixing;
    HoldingFB: HoldReaction;
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1; (* Initiate heating phase *)
            PhaseTimer(IN := TRUE, PT := MaxPhaseTime);
        END_IF;
    
    1: (* Heating Phase *)
        (* Call heating function block *)
        HeatingFB(TempSetpoint := ReactionTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HeatingFB.HeaterCmd;
        
        (* Transition to mixing when heating complete *)
        IF HeatingFB.HeatingComplete AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransitionDelay(IN := FALSE);
        ELSIF HeatingFB.HeatingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Mixing Phase *)
        (* Call mixing function block *)
        MixingFB(SpeedSetpoint := MixingSpeedSetpoint, 
                 CurrentSpeed := CurrentMixingSpeed, 
                 Tolerance := SpeedTolerance);
        MixerOn := MixingFB.MixerCmd;
        
        (* Maintain heater to stabilize temperature *)
        HeatingFB(TempSetpoint := ReactionTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HeatingFB.HeaterCmd;
        
        (* Transition to holding when mixing complete *)
        IF MixingFB.MixingComplete AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransitionDelay(IN := FALSE);
        ELSIF MixingFB.MixingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Holding Phase *)
        (* Call holding function block *)
        HoldingFB(HoldDuration := ReactionHoldTime, Timer := HoldTimer);
        
        (* Maintain heater and mixer *)
        HeatingFB(TempSetpoint := ReactionTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HoldingFB.HoldComplete ? FALSE : HeatingFB.HeaterCmd;
        MixingFB(SpeedSetpoint := MixingSpeedSetpoint, 
                 CurrentSpeed := CurrentMixingSpeed, 
                 Tolerance := SpeedTolerance);
        MixerOn := HoldingFB.HoldComplete ? FALSE : MixingFB.MixerCmd;
        
        (* Transition to complete when holding done *)
        IF HoldingFB.HoldComplete AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransitionDelay(IN := FALSE);
            ReactionComplete := TRUE;
        ELSIF HoldingFB.HoldComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Complete State *)
        (* Reset outputs *)
        HeaterOn := FALSE;
        MixerOn := FALSE;
        PhaseTimer(IN := FALSE);
        HoldTimer(IN := FALSE);
        
        (* Wait for reset or next batch *)
        IF ResetCommand THEN
            PhaseState := 0;
            ReactionComplete := FALSE;
        END_IF;
    
    5: (* Fault State *)
        (* Disable all outputs *)
        HeaterOn := FALSE;
        MixerOn := FALSE;
        PhaseTimer(IN := FALSE);
        HoldTimer(IN := FALSE);
        
        (* Require manual reset *)
        IF ResetCommand AND NOT EmergencyStop THEN
            PhaseState := 0;
            FaultState := FALSE;
        END_IF;
END_CASE;

(* Safety and Fault Monitoring *)
IF EmergencyStop THEN
    PhaseState := 5;
    FaultState := TRUE;
END_IF;

(* Process Parameter Monitoring *)
IF CurrentTemp > (ReactionTempSetpoint + 10.0) OR 
   CurrentTemp < (ReactionTempSetpoint - 10.0) AND PhaseState IN (1..3) THEN
    FaultState := TRUE;
    PhaseState := 5;
END_IF;

IF CurrentMixingSpeed > (MixingSpeedSetpoint + 100.0) AND PhaseState IN (2..3) THEN
    FaultState := TRUE;
    PhaseState := 5;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 5;
END_IF;

END_PROGRAM
