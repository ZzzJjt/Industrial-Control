(* ISA-88 Compliant Structured Text Program for Aspirin Production *)
(* Reaction Stage (Unit Procedure 1) *)
(* Conforms to IEC 61131-3 Standard *)

PROGRAM AspirinReactionControl
VAR
    (* ISA-88 Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: Charging, 2: Heating, 3: Mixing, 4: Holding, 5: Cooling, 6: Complete, 7: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters - Reaction *)
    ReactionTempSetpoint: REAL := 80.0; (* Target reaction temperature in °C *)
    TempTolerance: REAL := 2.0; (* Acceptable temperature deviation *)
    MixingSpeedSetpoint: REAL := 300.0; (* Mixer speed in RPM *)
    SpeedTolerance: REAL := 30.0; (* Acceptable speed deviation *)
    ReactionHoldTime: TIME := T#60m; (* Duration for holding reaction *)
    MaxPhaseTime: TIME := T#90m; (* Maximum duration for reaction phase *)
    PressureLimit: REAL := 1.2; (* Max pressure in bar, atmospheric ~1.0 bar *)
    
    (* Process Parameters - Crystallization *)
    CrystCoolingTemp: REAL := 20.0; (* Crystallization temperature *)
    CrystCoolingRate: REAL := 1.0; (* Max cooling rate in °C/min *)
    
    (* Process Parameters - Drying *)
    DryingTempSetpoint: REAL := 90.0; (* Drying temperature *)
    DryingTempTolerance: REAL := 1.0; (* Tighter tolerance for drying *)
    DryingTime: TIME := T#2h; (* Drying duration *)
    
    (* Process Inputs *)
    CurrentTemp: REAL; (* Current reactor temperature *)
    CurrentPressure: REAL; (* Current reactor pressure *)
    CurrentMixingSpeed: REAL; (* Current mixer speed *)
    MaterialCharged: BOOL; (* Material charging complete *)
    CrystCurrentTemp: REAL; (* Crystallizer temperature *)
    DryerCurrentTemp: REAL; (* Dryer temperature *)
    
    (* Process Outputs *)
    HeaterOn: BOOL; (* Reactor heater control *)
    MixerOn: BOOL; (* Reactor mixer control *)
    CoolingValve: BOOL; (* Reactor cooling control *)
    MaterialValve: BOOL; (* Material charging valve *)
    TransferPump: BOOL; (* Transfer to crystallizer *)
    CrystCoolerOn: BOOL; (* Crystallizer cooling control *)
    DryerHeaterOn: BOOL; (* Dryer heater control *)
    ReactionComplete: BOOL; (* Reaction phase completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Timer for overall phase duration *)
    HoldTimer: TON; (* Timer for reaction holding *)
    CrystTimer: TON; (* Timer for crystallization *)
    DryingTimer: TON; (* Timer for drying *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
END_VAR

(* Modular Function Blocks *)
FUNCTION_BLOCK ChargeMaterials
    VAR_INPUT
        Charged: BOOL;
    END_VAR
    VAR_OUTPUT
        ValveCmd: BOOL;
        ChargeComplete: BOOL;
    END_VAR
    (* Logic to charge materials *)
    ValveCmd := NOT Charged;
    ChargeComplete := Charged;
END_FUNCTION_BLOCK

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

FUNCTION_BLOCK StartCooling
    VAR_INPUT
        CurrentTemp: REAL;
        TargetTemp: REAL;
        CoolingRate: REAL;
    END_VAR
    VAR_OUTPUT
        CoolingCmd: BOOL;
        CoolingComplete: BOOL;
    END_VAR
    (* Logic to control cooling rate *)
    CoolingCmd := (CurrentTemp > TargetTemp);
    CoolingComplete := (CurrentTemp <= TargetTemp);
END_FUNCTION_BLOCK

FUNCTION_BLOCK ControlCrystallization
    VAR_INPUT
        CurrentTemp: REAL;
        TargetTemp: REAL;
        CoolingRate: REAL;
        Timer: TON;
    END_VAR
    VAR_OUTPUT
        CoolerCmd: BOOL;
        CrystComplete: BOOL;
    END_VAR
    (* Logic for crystallization *)
    CoolerCmd := (CurrentTemp > TargetTemp);
    Timer(IN := CoolerCmd, PT := T#30m);
    CrystComplete := (CurrentTemp <= TargetTemp) AND Timer.Q;
END_FUNCTION_BLOCK

FUNCTION_BLOCK ControlDrying
    VAR_INPUT
        TempSetpoint: REAL;
        CurrentTemp: REAL;
        Tolerance: REAL;
        Timer: TON;
        Duration: TIME;
    END_VAR
    VAR_OUTPUT
        HeaterCmd: BOOL;
        DryingComplete: BOOL;
    END_VAR
    (* Logic for drying at 90°C *)
    HeaterCmd := (CurrentTemp < (TempSetpoint - Tolerance));
    Timer(IN := (ABS(CurrentTemp - TempSetpoint) <= Tolerance), PT := Duration);
    DryingComplete := Timer.Q;
END_FUNCTION_BLOCK

(* Main Phase Control Logic *)
VAR
    ChargeFB: ChargeMaterials;
    HeatingFB: StartHeating;
    MixingFB: StartMixing;
    HoldingFB: HoldReaction;
    CoolingFB: StartCooling;
    CrystFB: ControlCrystallization;
    DryingFB: ControlDrying;
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1; (* Initiate charging *)
            PhaseTimer(IN := TRUE, PT := MaxPhaseTime);
        END_IF;
    
    1: (* Charging Phase *)
        (* Charge salicylic acid, acetic anhydride, sulfuric acid *)
        ChargeFB(Charged := MaterialCharged);
        MaterialValve := ChargeFB.ValveCmd;
        
        (* Transition to heating *)
        IF ChargeFB.ChargeComplete AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransitionDelay(IN := FALSE);
        ELSIF ChargeFB.ChargeComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Heating Phase *)
        (* Heat to 80°C *)
        HeatingFB(TempSetpoint := ReactionTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HeatingFB.HeaterCmd;
        
        (* Transition to mixing *)
        IF HeatingFB.HeatingComplete AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransitionDelay(IN := FALSE);
        ELSIF HeatingFB.HeatingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Mixing Phase *)
        (* Start mixing at 300 RPM *)
        MixingFB(SpeedSetpoint := MixingSpeedSetpoint, 
                 CurrentSpeed := CurrentMixingSpeed, 
                 Tolerance := SpeedTolerance);
        MixerOn := MixingFB.MixerCmd;
        
        (* Maintain temperature *)
        HeatingFB(TempSetpoint := ReactionTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HeatingFB.HeaterCmd;
        
        (* Transition to holding *)
        IF MixingFB.MixingComplete AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransitionDelay(IN := FALSE);
        ELSIF MixingFB.MixingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Holding Phase *)
        (* Hold reaction for 60 minutes *)
        HoldingFB(HoldDuration := ReactionHoldTime, Timer := HoldTimer);
        
        (* Maintain temperature and mixing *)
        HeatingFB(TempSetpoint := ReactionTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HoldingFB.HoldComplete ? FALSE : HeatingFB.HeaterCmd;
        MixingFB(SpeedSetpoint := MixingSpeedSetpoint, 
                 CurrentSpeed := CurrentMixingSpeed, 
                 Tolerance := SpeedTolerance);
        MixerOn := HoldingFB.HoldComplete ? FALSE : MixingFB.MixerCmd;
        
        (* Transition to cooling *)
        IF HoldingFB.HoldComplete AND TransitionDelay.Q THEN
            PhaseState := 5;
            TransitionDelay(IN := FALSE);
        ELSIF HoldingFB.HoldComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    5: (* Cooling Phase *)
        (* Cool to 20°C for crystallization *)
        CoolingFB(CurrentTemp := CurrentTemp, 
                  TargetTemp := CrystCoolingTemp, 
                  CoolingRate := CrystCoolingRate);
        CoolingValve := CoolingFB.CoolingCmd;
        
        (* Transfer to crystallizer *)
        IF CoolingFB.CoolingComplete AND TransitionDelay.Q THEN
            TransferPump := TRUE;
            PhaseState := 6;
            TransitionDelay(IN := FALSE);
        ELSIF CoolingFB.CoolingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    6: (* Complete State *)
        (* Reset outputs and signal completion *)
        HeaterOn := FALSE;
        MixerOn := FALSE;
        CoolingValve := FALSE;
        MaterialValve := FALSE;
        TransferPump := FALSE;
        PhaseTimer(IN := FALSE);
        HoldTimer(IN := FALSE);
        ReactionComplete := TRUE;
        
        (* Crystallization control *)
        CrystFB(CurrentTemp := CrystCurrentTemp, 
                TargetTemp := CrystCoolingTemp, 
                CoolingRate := CrystCoolingRate, 
                Timer := CrystTimer);
        CrystCoolerOn := CrystFB.CoolerCmd;
        
        (* Drying control *)
        IF CrystFB.CrystComplete THEN
            DryingFB(TempSetpoint := DryingTempSetpoint, 
                     CurrentTemp := DryerCurrentTemp, 
                     Tolerance := DryingTempTolerance, 
                     Timer := DryingTimer, 
                     Duration := DryingTime);
            DryerHeaterOn := DryingFB.HeaterCmd;
        END_IF;
        
        (* Reset for next batch *)
        IF ResetCommand AND DryingFB.DryingComplete THEN
            PhaseState := 0;
            ReactionComplete := FALSE;
            CrystCoolerOn := FALSE;
            DryerHeaterOn := FALSE;
        END_IF;
    
    7: (* Fault State *)
        (* Disable all outputs *)
        HeaterOn := FALSE;
        MixerOn := FALSE;
        CoolingValve := FALSE;
        MaterialValve := FALSE;
        TransferPump := FALSE;
        CrystCoolerOn := FALSE;
        DryerHeaterOn := FALSE;
        PhaseTimer(IN := FALSE);
        HoldTimer(IN := FALSE);
        CrystTimer(IN := FALSE);
        DryingTimer(IN := FALSE);
        
        (* Require manual reset *)
        IF ResetCommand AND NOT EmergencyStop THEN
            PhaseState := 0;
            FaultState := FALSE;
        END_IF;
END_CASE;

(* Safety and Fault Monitoring *)
IF EmergencyStop THEN
    PhaseState := 7;
    FaultState := TRUE;
END_IF;

(* Reaction Parameter Monitoring *)
IF CurrentTemp > (ReactionTempSetpoint + 10.0) OR 
   CurrentTemp < (ReactionTempSetpoint - 10.0) AND PhaseState IN (2..4) THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

IF CurrentMixingSpeed > (MixingSpeedSetpoint + 100.0) AND PhaseState IN (3..4) THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

IF CurrentPressure > PressureLimit AND PhaseState IN (1..5) THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

(* Dryer Temperature Monitoring *)
IF DryerCurrentTemp > (DryingTempSetpoint + 5.0) AND DryingFB.HeaterCmd THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

END_PROGRAM
