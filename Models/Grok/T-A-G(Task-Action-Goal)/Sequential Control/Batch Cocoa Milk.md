(* ISA-88 Compliant Structured Text Program for Cocoa Milk Production *)
(* Unit Procedure: Mixing Stage *)
(* Conforms to IEC 61131-3 Standard *)

PROGRAM CocoaMilkMixingControl
VAR
    (* ISA-88 Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: Charging, 2: Heating, 3: Blending, 4: Transfer, 5: Complete, 6: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters *)
    BatchSize: REAL := 100.0; (* Total batch size in kg *)
    MilkQty: REAL := 70.0; (* Milk quantity in kg *)
    WaterQty: REAL := 20.0; (* Water quantity in kg *)
    SugarQty: REAL := 8.0; (* Liquid sugar quantity in kg *)
    CocoaQty: REAL := 2.0; (* Cocoa powder quantity in kg *)
    TempSetpoint: REAL := 70.0; (* Target heating temperature in °C *)
    TempTolerance: REAL := 2.0; (* Acceptable temperature deviation *)
    MixingSpeedSetpoint: REAL := 200.0; (* Mixer speed in RPM *)
    SpeedTolerance: REAL := 20.0; (* Acceptable speed deviation *)
    MixingDuration: TIME := T#10m; (* Blending duration *)
    MaxPhaseTime: TIME := T#30m; (* Maximum duration for mixing phase *)
    
    (* Process Inputs *)
    CurrentTemp: REAL; (* Current tank temperature *)
    CurrentMixingSpeed: REAL; (* Current mixer speed *)
    MilkCharged: BOOL; (* Milk charging complete *)
    WaterCharged: BOOL; (* Water charging complete *)
    SugarCharged: BOOL; (* Liquid sugar charging complete *)
    CocoaCharged: BOOL; (* Cocoa powder charging complete *)
    
    (* Process Outputs *)
    MilkValve: BOOL; (* Milk inlet valve *)
    WaterValve: BOOL; (* Water inlet valve *)
    SugarValve: BOOL; (* Liquid sugar inlet valve *)
    CocoaValve: BOOL; (* Cocoa powder inlet valve *)
    HeaterOn: BOOL; (* Heater control *)
    MixerOn: BOOL; (* Mixer control *)
    TransferPump: BOOL; (* Transfer to holding tank *)
    MixingComplete: BOOL; (* Mixing phase completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Timer for overall phase duration *)
    MixingTimer: TON; (* Timer for blending phase *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
END_VAR

(* Modular Function Blocks *)
FUNCTION_BLOCK ChargeIngredients
    VAR_INPUT
        MilkCharged: BOOL;
        WaterCharged: BOOL;
        SugarCharged: BOOL;
        CocoaCharged: BOOL;
    END_VAR
    VAR_OUTPUT
        MilkValveCmd: BOOL;
        WaterValveCmd: BOOL;
        SugarValveCmd: BOOL;
        CocoaValveCmd: BOOL;
        ChargeComplete: BOOL;
    END_VAR
    (* Logic to charge ingredients sequentially *)
    MilkValveCmd := NOT MilkCharged;
    WaterValveCmd := MilkCharged AND NOT WaterCharged;
    SugarValveCmd := MilkCharged AND WaterCharged AND NOT SugarCharged;
    CocoaValveCmd := MilkCharged AND WaterCharged AND SugarCharged AND NOT CocoaCharged;
    ChargeComplete := MilkCharged AND WaterCharged AND SugarCharged AND CocoaCharged;
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

FUNCTION_BLOCK StartBlending
    VAR_INPUT
        SpeedSetpoint: REAL;
        CurrentSpeed: REAL;
        Tolerance: REAL;
        Duration: TIME;
        Timer: TON;
    END_VAR
    VAR_OUTPUT
        MixerCmd: BOOL;
        BlendingComplete: BOOL;
    END_VAR
    (* Logic to start and maintain blending *)
    MixerCmd := (CurrentSpeed < (SpeedSetpoint - Tolerance));
    Timer(IN := (ABS(CurrentSpeed - SpeedSetpoint) <= Tolerance), PT := Duration);
    BlendingComplete := Timer.Q;
END_FUNCTION_BLOCK

(* Main Phase Control Logic *)
VAR
    ChargeFB: ChargeIngredients;
    HeatingFB: StartHeating;
    BlendingFB: StartBlending;
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1; (* Initiate charging *)
            PhaseTimer(IN := TRUE, PT := MaxPhaseTime);
        END_IF;
    
    1: (* Charging Phase *)
        (* Charge milk, water, liquid sugar, cocoa sequentially *)
        ChargeFB(MilkCharged := MilkCharged, 
                 WaterCharged := WaterCharged, 
                 SugarCharged := SugarCharged, 
                 CocoaCharged := CocoaCharged);
        MilkValve := ChargeFB.MilkValveCmd;
        WaterValve := ChargeFB.WaterValveCmd;
        SugarValve := ChargeFB.SugarValveCmd;
        CocoaValve := ChargeFB.CocoaValveCmd;
        
        (* Transition to heating *)
        IF ChargeFB.ChargeComplete AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransitionDelay(IN := FALSE);
        ELSIF ChargeFB.ChargeComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Heating Phase *)
        (* Heat to 70°C *)
        HeatingFB(TempSetpoint := TempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HeatingFB.HeaterCmd;
        
        (* Start mixer at low speed to prevent settling *)
        MixerOn := TRUE;
        IF CurrentMixingSpeed < (MixingSpeedSetpoint * 0.5) THEN
            MixerOn := TRUE;
        END_IF;
        
        (* Transition to blending *)
        IF HeatingFB.HeatingComplete AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransitionDelay(IN := FALSE);
        ELSIF HeatingFB.HeatingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Blending Phase *)
        (* Blend at 200 RPM for 10 minutes *)
        BlendingFB(SpeedSetpoint := MixingSpeedSetpoint, 
                   CurrentSpeed := CurrentMixingSpeed, 
                   Tolerance := SpeedTolerance, 
                   Duration := MixingDuration, 
                   Timer := MixingTimer);
        MixerOn := BlendingFB.MixerCmd;
        
        (* Maintain temperature *)
        HeatingFB(TempSetpoint := TempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := BlendingFB.BlendingComplete ? FALSE : HeatingFB.HeaterCmd;
        
        (* Transition to transfer *)
        IF BlendingFB.BlendingComplete AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransitionDelay(IN := FALSE);
        ELSIF BlendingFB.BlendingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Transfer Phase *)
        (* Transfer to holding tank *)
        TransferPump := TRUE;
        
        (* Complete phase *)
        IF TransitionDelay.Q THEN
            PhaseState := 5;
            TransitionDelay(IN := FALSE);
            MixingComplete := TRUE;
        ELSE
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    5: (* Complete State *)
        (* Reset outputs *)
        MilkValve := FALSE;
        WaterValve := FALSE;
        SugarValve := FALSE;
        CocoaValve := FALSE;
        HeaterOn := FALSE;
        MixerOn := FALSE;
        TransferPump := FALSE;
        PhaseTimer(IN := FALSE);
        MixingTimer(IN := FALSE);
        
        (* Wait for reset or next batch *)
        IF ResetCommand THEN
            PhaseState := 0;
            MixingComplete := FALSE;
        END_IF;
    
    6: (* Fault State *)
        (* Disable all outputs *)
        MilkValve := FALSE;
        WaterValve := FALSE;
        SugarValve := FALSE;
        CocoaValve := FALSE;
        HeaterOn := FALSE;
        MixerOn := FALSE;
        TransferPump := FALSE;
        PhaseTimer(IN := FALSE);
        MixingTimer(IN := FALSE);
        
        (* Require manual reset *)
        IF ResetCommand AND NOT EmergencyStop THEN
            PhaseState := 0;
            FaultState := FALSE;
        END_IF;
END_CASE;

(* Safety and Fault Monitoring *)
IF EmergencyStop THEN
    PhaseState := 6;
    FaultState := TRUE;
END_IF;

(* Process Parameter Monitoring *)
IF CurrentTemp > (TempSetpoint + 10.0) OR 
   CurrentTemp < (TempSetpoint - 10.0) AND PhaseState IN (2..3) THEN
    FaultState := TRUE;
    PhaseState := 6;
END_IF;

IF CurrentMixingSpeed > (MixingSpeedSetpoint + 50.0) AND PhaseState = 3 THEN
    FaultState := TRUE;
    PhaseState := 6;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 6;
END_IF;

END_PROGRAM
