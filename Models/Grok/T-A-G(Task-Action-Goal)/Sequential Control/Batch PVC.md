(* ISA-88 Compliant Structured Text Program for PVC Production *)
(* Unit Procedures: Polymerize, Decover, Dry *)
(* Conforms to IEC 61131-3 Standard *)

PROGRAM PVCPolymerizationControl
VAR
    (* ISA-88 Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: Evacuating, 2: Charging, 3: Polymerizing, 4: Decovering, 5: Drying, 6: Complete, 7: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters - Polymerization *)
    TempMinSetpoint: REAL := 55.0; (* Minimum polymerization temperature in °C *)
    TempMaxSetpoint: REAL := 60.0; (* Maximum polymerization temperature in °C *)
    MixingSpeedSetpoint: REAL := 150.0; (* Mixer speed in RPM *)
    SpeedTolerance: REAL := 15.0; (* Acceptable speed deviation *)
    PressureThreshold: REAL := 8.0; (* Pressure drop threshold in bar *)
    PolymerizationTime: TIME := T#6h; (* Polymerization duration *)
    VacuumPressure: REAL := 0.1; (* Target vacuum pressure in bar *)
    MaxPolymerizeTime: TIME := T#8h; (* Maximum polymerization phase duration *)
    
    (* Process Parameters - Drying *)
    DryingTempSetpoint: REAL := 80.0; (* Drying temperature in °C *)
    DryingTempTolerance: REAL := 2.0; (* Acceptable drying temperature deviation *)
    DryingTime: TIME := T#4h; (* Drying duration *)
    
    (* Process Inputs *)
    CurrentTemp: REAL; (* Current reactor temperature *)
    CurrentPressure: REAL; (* Current reactor pressure *)
    CurrentMixingSpeed: REAL; (* Current mixer speed *)
    WaterCharged: BOOL; (* Demineralized water charged *)
    SurfactantCharged: BOOL; (* Surfactants charged *)
    VCMCharged: BOOL; (* VCM charged *)
    CatalystCharged: BOOL; (* Catalyst charged *)
    DryerCurrentTemp: REAL; (* Dryer temperature *)
    
    (* Process Outputs *)
    VacuumPump: BOOL; (* Vacuum pump for reactor evacuation *)
    NitrogenValve: BOOL; (* Nitrogen purge valve *)
    WaterValve: BOOL; (* Demineralized water inlet valve *)
    SurfactantValve: BOOL; (* Surfactant inlet valve *)
    VCMValve: BOOL; (* VCM inlet valve *)
    CatalystValve: BOOL; (* Catalyst inlet valve *)
    HeaterOn: BOOL; (* Reactor heater control *)
    CoolerOn: BOOL; (* Reactor cooling control *)
    MixerOn: BOOL; (* Reactor mixer control *)
    DecoverValve: BOOL; (* Decover system valve *)
    TransferPump: BOOL; (* Transfer to dryer *)
    DryerHeaterOn: BOOL; (* Dryer heater control *)
    BatchComplete: BOOL; (* Batch completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Timer for overall phase duration *)
    PolymerizationTimer: TON; (* Timer for polymerization *)
    DryingTimer: TON; (* Timer for drying *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
END_VAR

(* Modular Function Blocks *)
FUNCTION_BLOCK EvacuateReactor
    VAR_INPUT
        CurrentPressure: REAL;
        VacuumTarget: REAL;
    END_VAR
    VAR_OUTPUT
        VacuumPumpCmd: BOOL;
        NitrogenValveCmd: BOOL;
        EvacuationComplete: BOOL;
    END_VAR
    (* Logic to evacuate reactor *)
    VacuumPumpCmd := (CurrentPressure > VacuumTarget);
    NitrogenValveCmd := FALSE; (* Nitrogen purge after vacuum *)
    EvacuationComplete := (CurrentPressure <= VacuumTarget);
END_FUNCTION_BLOCK

FUNCTION_BLOCK AddDemineralizedWater
    VAR_INPUT
        WaterCharged: BOOL;
    END_VAR
    VAR_OUTPUT
        WaterValveCmd: BOOL;
        WaterChargeComplete: BOOL;
    END_VAR
    (* Logic to charge demineralized water *)
    WaterValveCmd := NOT WaterCharged;
    WaterChargeComplete := WaterCharged;
END_FUNCTION_BLOCK

FUNCTION_BLOCK AddSurfactants
    VAR_INPUT
        SurfactantCharged: BOOL;
    END_VAR
    VAR_OUTPUT
        SurfactantValveCmd: BOOL;
        SurfactantChargeComplete: BOOL;
    END_VAR
    (* Logic to charge surfactants *)
    SurfactantValveCmd := NOT SurfactantCharged;
    SurfactantChargeComplete := SurfactantCharged;
END_FUNCTION_BLOCK

FUNCTION_BLOCK AddVCM
    VAR_INPUT
        VCMCharged: BOOL;
    END_VAR
    VAR_OUTPUT
        VCMValveCmd: BOOL;
        VCMChargeComplete: BOOL;
    END_VAR
    (* Logic to charge VCM *)
    VCMValveCmd := NOT VCMCharged;
    VCMChargeComplete := VCMCharged;
END_FUNCTION_BLOCK

FUNCTION_BLOCK AddCatalyst
    VAR_INPUT
        CatalystCharged: BOOL;
    END_VAR
    VAR_OUTPUT
        CatalystValveCmd: BOOL;
        CatalystChargeComplete: BOOL;
    END_VAR
    (* Logic to charge catalyst *)
    CatalystValveCmd := NOT CatalystCharged;
    CatalystChargeComplete := CatalystCharged;
END_FUNCTION_BLOCK

FUNCTION_BLOCK ControlPolymerization
    VAR_INPUT
        CurrentTemp: REAL;
        TempMin: REAL;
        TempMax: REAL;
        CurrentPressure: REAL;
        PressureThreshold: REAL;
        CurrentSpeed: REAL;
        SpeedSetpoint: REAL;
        SpeedTolerance: REAL;
        Timer: TON;
        Duration: TIME;
    END_VAR
    VAR_OUTPUT
        HeaterCmd: BOOL;
        CoolerCmd: BOOL;
        MixerCmd: BOOL;
        PolymerizationComplete: BOOL;
    END_VAR
    (* Logic to control polymerization *)
    HeaterCmd := (CurrentTemp < TempMin);
    CoolerCmd := (CurrentTemp > TempMax);
    MixerCmd := (CurrentSpeed < (SpeedSetpoint - SpeedTolerance));
    Timer(IN := (ABS(CurrentSpeed - SpeedSetpoint) <= SpeedTolerance) AND
                (CurrentTemp >= TempMin) AND (CurrentTemp <= TempMax), 
          PT := Duration);
    PolymerizationComplete := Timer.Q OR (CurrentPressure < PressureThreshold);
END_FUNCTION_BLOCK

FUNCTION_BLOCK ControlDecover
    VAR_INPUT
        TransferReady: BOOL;
    END_VAR
    VAR_OUTPUT
        DecoverValveCmd: BOOL;
        TransferPumpCmd: BOOL;
        DecoverComplete: BOOL;
    END_VAR
    (* Logic to handle decover and transfer *)
    DecoverValveCmd := TransferReady;
    TransferPumpCmd := TransferReady;
    DecoverComplete := TransferReady;
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
    (* Logic for drying at 80°C *)
    HeaterCmd := (CurrentTemp < (TempSetpoint - Tolerance));
    Timer(IN := (ABS(CurrentTemp - TempSetpoint) <= Tolerance), PT := Duration);
    DryingComplete := Timer.Q;
END_FUNCTION_BLOCK

(* Main Phase Control Logic *)
VAR
    EvacuateFB: EvacuateReactor;
    WaterFB: AddDemineralizedWater;
    SurfactantFB: AddSurfactants;
    VCMFB: AddVCM;
    CatalystFB: AddCatalyst;
    PolymerizeFB: ControlPolymerization;
    DecoverFB: ControlDecover;
    DryingFB: ControlDrying;
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1; (* Initiate evacuation *)
            PhaseTimer(IN := TRUE, PT := MaxPolymerizeTime);
        END_IF;
    
    1: (* Evacuating Phase *)
        (* Evacuate reactor to <0.1 bar *)
        EvacuateFB(CurrentPressure := CurrentPressure, VacuumTarget := VacuumPressure);
        VacuumPump := EvacuateFB.VacuumPumpCmd;
        NitrogenValve := EvacuateFB.NitrogenValveCmd;
        
        (* Transition to charging *)
        IF EvacuateFB.EvacuationComplete AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransitionDelay(IN := FALSE);
        ELSIF EvacuateFB.EvacuationComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Charging Phase *)
        (* Charge demineralized water *)
        WaterFB(WaterCharged := WaterCharged);
        WaterValve := WaterFB.WaterValveCmd;
        
        (* Charge surfactants *)
        IF WaterFB.WaterChargeComplete THEN
            SurfactantFB(SurfactantCharged := SurfactantCharged);
            SurfactantValve := SurfactantFB.SurfactantValveCmd;
        END_IF;
        
        (* Charge VCM *)
        IF SurfactantFB.SurfactantChargeComplete THEN
            VCMFB(VCMCharged := VCMCharged);
            VCMValve := VCMFB.VCMValveCmd;
        END_IF;
        
        (* Charge catalyst *)
        IF VCMFB.VCMChargeComplete THEN
            CatalystFB(CatalystCharged := CatalystCharged);
            CatalystValve := CatalystFB.CatalystValveCmd;
        END_IF;
        
        (* Transition to polymerizing *)
        IF CatalystFB.CatalystChargeComplete AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransitionDelay(IN := FALSE);
        ELSIF CatalystFB.CatalystChargeComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Polymerizing Phase *)
        (* Control polymerization at 55–60°C, 150 RPM *)
        PolymerizeFB(CurrentTemp := CurrentTemp, 
                     TempMin := TempMinSetpoint, 
                     TempMax := TempMaxSetpoint, 
                     CurrentPressure := CurrentPressure, 
                     PressureThreshold := PressureThreshold, 
                     CurrentSpeed := CurrentMixingSpeed, 
                     SpeedSetpoint := MixingSpeedSetpoint, 
                     SpeedTolerance := SpeedTolerance, 
                     Timer := PolymerizationTimer, 
                     Duration := PolymerizationTime);
        HeaterOn := PolymerizeFB.HeaterCmd;
        CoolerOn := PolymerizeFB.CoolerCmd;
        MixerOn := PolymerizeFB.MixerCmd;
        
        (* Transition to decovering *)
        IF PolymerizeFB.PolymerizationComplete AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransitionDelay(IN := FALSE);
        ELSIF PolymerizeFB.PolymerizationComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Decovering Phase *)
        (* Release or transfer material *)
        DecoverFB(TransferReady := TRUE); (* Assume system is ready *)
        DecoverValve := DecoverFB.DecoverValveCmd;
        TransferPump := DecoverFB.TransferPumpCmd;
        
        (* Transition to drying *)
        IF DecoverFB.DecoverComplete AND TransitionDelay.Q THEN
            PhaseState := 5;
            TransitionDelay(IN := FALSE);
        ELSIF DecoverFB.DecoverComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    5: (* Drying Phase *)
        (* Dry at 80°C for 4 hours *)
        DryingFB(TempSetpoint := DryingTempSetpoint, 
                 CurrentTemp := DryerCurrentTemp, 
                 Tolerance := DryingTempTolerance, 
                 Timer := DryingTimer, 
                 Duration := DryingTime);
        DryerHeaterOn := DryingFB.HeaterCmd;
        
        (* Transition to complete *)
        IF DryingFB.DryingComplete AND TransitionDelay.Q THEN
            PhaseState := 6;
            TransitionDelay(IN := FALSE);
            BatchComplete := TRUE;
        ELSIF DryingFB.DryingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    6: (* Complete State *)
        (* Reset outputs *)
        VacuumPump := FALSE;
        NitrogenValve := FALSE;
        WaterValve := FALSE;
        SurfactantValve := FALSE;
        VCMValve := FALSE;
        CatalystValve := FALSE;
        HeaterOn := FALSE;
        CoolerOn := FALSE;
        MixerOn := FALSE;
        DecoverValve := FALSE;
        TransferPump := FALSE;
        DryerHeaterOn := FALSE;
        PhaseTimer(IN := FALSE);
        PolymerizationTimer(IN := FALSE);
        DryingTimer(IN := FALSE);
        
        (* Wait for reset *)
        IF ResetCommand THEN
            PhaseState := 0;
            BatchComplete := FALSE;
        END_IF;
    
    7: (* Fault State *)
        (* Disable all outputs *)
        VacuumPump := FALSE;
        NitrogenValve := FALSE;
        WaterValve := FALSE;
        SurfactantValve := FALSE;
        VCMValve := FALSE;
        CatalystValve := FALSE;
        HeaterOn := FALSE;
        CoolerOn := FALSE;
        MixerOn := FALSE;
        DecoverValve := FALSE;
        TransferPump := FALSE;
        DryerHeaterOn := FALSE;
        PhaseTimer(IN := FALSE);
        PolymerizationTimer(IN := FALSE);
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

(* Polymerization Parameter Monitoring *)
IF CurrentTemp > (TempMaxSetpoint + 5.0) OR 
   CurrentTemp < (TempMinSetpoint - 5.0) AND PhaseState = 3 THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

IF CurrentMixingSpeed > (MixingSpeedSetpoint + 50.0) AND PhaseState = 3 THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

IF CurrentPressure > 12.0 AND PhaseState IN (1..3) THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

(* Dryer Temperature Monitoring *)
IF DryerCurrentTemp > (DryingTempSetpoint + 5.0) AND PhaseState = 5 THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 7;
END_IF;

END_PROGRAM
