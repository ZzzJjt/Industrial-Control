(* ISA-88 Compliant Structured Text Program for Polyethylene Production *)
(* Full-Cycle Batch Process *)
(* Conforms to IEC 61131-3 Standard *)

PROGRAM PolyethyleneProductionControl
VAR
    (* ISA-88 Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: Prep, 2: Polymerizing, 3: Quenching, 4: Drying, 5: Pelletizing, 6: QualityControl, 7: Packaging, 8: Complete, 9: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters *)
    (* Material Preparation *)
    EthyleneQty: REAL := 1000.0; (* Ethylene monomer in kg *)
    CatalystQty: REAL := 10.0; (* Catalyst in kg *)
    SolventQty: REAL := 500.0; (* Solvent in kg *)
    
    (* Polymerization *)
    PolyTempSetpoint: REAL := 200.0; (* Polymerization temperature in °C *)
    PolyTempTolerance: REAL := 5.0; (* Temperature deviation *)
    PolyPressureSetpoint: REAL := 1000.0; (* Pressure in bar *)
    PolyPressureTolerance: REAL := 50.0; (* Pressure deviation *)
    PolyMixingSpeed: REAL := 200.0; (* Mixer speed in RPM *)
    PolySpeedTolerance: REAL := 20.0; (* Speed deviation *)
    PolyDuration: TIME := T#4h; (* Polymerization duration *)
    MaxPolyTime: TIME := T#6h; (* Maximum polymerization phase duration *)
    
    (* Quenching *)
    QuenchTempSetpoint: REAL := 50.0; (* Quench temperature in °C *)
    QuenchTempTolerance: REAL := 2.0; (* Temperature deviation *)
    
    (* Drying *)
    DryTempSetpoint: REAL := 80.0; (* Drying temperature in °C *)
    DryTempTolerance: REAL := 2.0; (* Temperature deviation *)
    DryDuration: TIME := T#3h; (* Drying duration *)
    
    (* Pelletizing *)
    PelletTempSetpoint: REAL := 150.0; (* Pelletizer temperature in °C *)
    PelletTempTolerance: REAL := 5.0; (* Temperature deviation *)
    PelletSpeedSetpoint: REAL := 100.0; (* Pelletizer speed in RPM *)
    PelletSpeedTolerance: REAL := 10.0; (* Speed deviation *)
    
    (* Quality Control *)
    MeltIndexMax: REAL := 2.0; (* Max melt index in g/10 min *)
    DensityMin: REAL := 0.94; (* Min density in g/cm³ *)
    DensityMax: REAL := 0.96; (* Max density in g/cm³ *)
    
    (* Process Inputs *)
    CurrentPrepTemp: REAL; (* Prep tank temperature *)
    CurrentPolyTemp: REAL; (* Reactor temperature *)
    CurrentPolyPressure: REAL; (* Reactor pressure *)
    CurrentPolySpeed: REAL; (* Reactor mixer speed *)
    CurrentQuenchTemp: REAL; (* Quench tank temperature *)
    CurrentDryTemp: REAL; (* Dryer temperature *)
    CurrentPelletTemp: REAL; (* Pelletizer temperature *)
    CurrentPelletSpeed: REAL; (* Pelletizer speed *)
    EthyleneCharged: BOOL; (* Ethylene charged *)
    CatalystCharged: BOOL; (* Catalyst charged *)
    SolventCharged: BOOL; (* Solvent charged *)
    MeltIndex: REAL; (* Measured melt index *)
    Density: REAL; (* Measured density *)
    UnitReady: ARRAY[1..7] OF BOOL; (* Unit readiness signals *)
    
    (* Process Outputs *)
    EthyleneValve: BOOL; (* Ethylene inlet valve *)
    CatalystValve: BOOL; (* Catalyst inlet valve *)
    SolventValve: BOOL; (* Solvent inlet valve *)
    NitrogenValve: BOOL; (* Nitrogen purge valve *)
    PolyHeaterOn: BOOL; (* Reactor heater *)
    PolyCoolerOn: BOOL; (* Reactor cooler *)
    PolyMixerOn: BOOL; (* Reactor mixer *)
    QuenchCoolerOn: BOOL; (* Quench tank cooler *)
    DryHeaterOn: BOOL; (* Dryer heater *)
    PelletHeaterOn: BOOL; (* Pelletizer heater *)
    PelletMotorOn: BOOL; (* Pelletizer motor *)
    TransferPump: ARRAY[1..6] OF BOOL; (* Transfer pumps between units *)
    PackageConveyor: BOOL; (* Packaging conveyor *)
    BatchComplete: BOOL; (* Batch completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Overall phase timer *)
    PolyTimer: TON; (* Polymerization timer *)
    DryTimer: TON; (* Drying timer *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
END_VAR

(* Modular Function Blocks *)
FUNCTION_BLOCK AddMaterial
    VAR_INPUT
        MaterialCharged: BOOL;
    END_VAR
    VAR_OUTPUT
        ValveCmd: BOOL;
        ChargeComplete: BOOL;
    END_VAR
    (* Logic to charge material *)
    ValveCmd := NOT MaterialCharged;
    ChargeComplete := MaterialCharged;
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartReaction
    VAR_INPUT
        CurrentTemp: REAL;
        TempSetpoint: REAL;
        TempTolerance: REAL;
        CurrentPressure: REAL;
        PressureSetpoint: REAL;
        PressureTolerance: REAL;
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
        ReactionComplete: BOOL;
    END_VAR
    (* Logic to control polymerization *)
    HeaterCmd := (CurrentTemp < (TempSetpoint - TempTolerance));
    CoolerCmd := (CurrentTemp > (TempSetpoint + TempTolerance));
    MixerCmd := (CurrentSpeed < (SpeedSetpoint - SpeedTolerance));
    Timer(IN := (ABS(CurrentTemp - TempSetpoint) <= TempTolerance) AND
                (ABS(CurrentPressure - PressureSetpoint) <= PressureTolerance) AND
                (ABS(CurrentSpeed - SpeedSetpoint) <= SpeedTolerance), 
          PT := Duration);
    ReactionComplete := Timer.Q;
END_FUNCTION_BLOCK

FUNCTION_BLOCK BeginQuenching
    VAR_INPUT
        CurrentTemp: REAL;
        TempSetpoint: REAL;
        TempTolerance: REAL;
    END_VAR
    VAR_OUTPUT
        CoolerCmd: BOOL;
        QuenchComplete: BOOL;
    END_VAR
    (* Logic to quench polymer *)
    CoolerCmd := (CurrentTemp > TempSetpoint);
    QuenchComplete := (ABS(CurrentTemp - TempSetpoint) <= TempTolerance);
END_FUNCTION_BLOCK

FUNCTION_BLOCK BeginDrying
    VAR_INPUT
        TempSetpoint: REAL;
        CurrentTemp: REAL;
        TempTolerance: REAL;
        Timer: TON;
        Duration: TIME;
    END_VAR
    VAR_OUTPUT
        HeaterCmd: BOOL;
        DryingComplete: BOOL;
    END_VAR
    (* Logic for drying *)
    HeaterCmd := (CurrentTemp < (TempSetpoint - TempTolerance));
    Timer(IN := (ABS(CurrentTemp - TempSetpoint) <= TempTolerance), PT := Duration);
    DryingComplete := Timer.Q;
END_FUNCTION_BLOCK

FUNCTION_BLOCK BeginPelletizing
    VAR_INPUT
        TempSetpoint: REAL;
        CurrentTemp: REAL;
        TempTolerance: REAL;
        SpeedSetpoint: REAL;
        CurrentSpeed: REAL;
        SpeedTolerance: REAL;
    END_VAR
    VAR_OUTPUT
        HeaterCmd: BOOL;
        MotorCmd: BOOL;
        PelletizingComplete: BOOL;
    END_VAR
    (* Logic for pelletizing *)
    HeaterCmd := (CurrentTemp < (TempSetpoint - TempTolerance));
    MotorCmd := (CurrentSpeed < (SpeedSetpoint - SpeedTolerance));
    PelletizingComplete := (ABS(CurrentTemp - TempSetpoint) <= TempTolerance) AND
                          (ABS(CurrentSpeed - SpeedSetpoint) <= SpeedTolerance);
END_FUNCTION_BLOCK

FUNCTION_BLOCK PerformQualityControl
    VAR_INPUT
        MeltIndex: REAL;
        MeltIndexMax: REAL;
        Density: REAL;
        DensityMin: REAL;
        DensityMax: REAL;
    END_VAR
    VAR_OUTPUT
        QualityPassed: BOOL;
    END_VAR
    (* Logic for quality control *)
    QualityPassed := (MeltIndex <= MeltIndexMax) AND
                     (Density >= DensityMin) AND (Density <= DensityMax);
END_FUNCTION_BLOCK

FUNCTION_BLOCK BeginPackaging
    VAR_INPUT
        QualityPassed: BOOL;
    END_VAR
    VAR_OUTPUT
        ConveyorCmd: BOOL;
        PackagingComplete: BOOL;
    END_VAR
    (* Logic for packaging *)
    ConveyorCmd := QualityPassed;
    PackagingComplete := QualityPassed;
END_FUNCTION_BLOCK

(* Main Phase Control Logic *)
VAR
    EthyleneFB: AddMaterial;
    CatalystFB: AddMaterial;
    SolventFB: AddMaterial;
    ReactionFB: StartReaction;
    QuenchFB: BeginQuenching;
    DryFB: BeginDrying;
    PelletizeFB: BeginPelletizing;
    QualityFB: PerformQualityControl;
    PackageFB: BeginPackaging;
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1; (* Initiate material preparation *)
            PhaseTimer(IN := TRUE, PT := T#24h); (* Max batch duration *)
        END_IF;
    
    1: (* Raw Material Preparation *)
        (* Charge ethylene, catalyst, solvent *)
        EthyleneFB(MaterialCharged := EthyleneCharged);
        EthyleneValve := EthyleneFB.ValveCmd;
        
        IF EthyleneFB.ChargeComplete THEN
            CatalystFB(MaterialCharged := CatalystCharged);
            CatalystValve := CatalystFB.ValveCmd;
        END_IF;
        
        IF CatalystFB.ChargeComplete THEN
            SolventFB(MaterialCharged := SolventCharged);
            SolventValve := SolventFB.ValveCmd;
        END_IF;
        
        (* Transition to polymerization *)
        IF SolventFB.ChargeComplete AND UnitReady[2] AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransferPump[1] := TRUE; (* Prep to reactor *)
            TransitionDelay(IN := FALSE);
        ELSIF SolventFB.ChargeComplete AND UnitReady[2] THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Polymerization *)
        (* Control reaction at 200°C, 1000 bar, 200 RPM *)
        ReactionFB(CurrentTemp := CurrentPolyTemp, 
                   TempSetpoint := PolyTempSetpoint, 
                   TempTolerance := PolyTempTolerance, 
                   CurrentPressure := CurrentPolyPressure, 
                   PressureSetpoint := PolyPressureSetpoint, 
                   PressureTolerance := PolyPressureTolerance, 
                   CurrentSpeed := CurrentPolySpeed, 
                   SpeedSetpoint := PolyMixingSpeed, 
                   SpeedTolerance := PolySpeedTolerance, 
                   Timer := PolyTimer, 
                   Duration := PolyDuration);
        PolyHeaterOn := ReactionFB.HeaterCmd;
        PolyCoolerOn := ReactionFB.CoolerCmd;
        PolyMixerOn := ReactionFB.MixerCmd;
        
        (* Transition to quenching *)
        IF ReactionFB.ReactionComplete AND UnitReady[3] AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransferPump[2] := TRUE; (* Reactor to quench *)
            TransitionDelay(IN := FALSE);
        ELSIF ReactionFB.ReactionComplete AND UnitReady[3] THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Quenching *)
        (* Cool to 50°C *)
        QuenchFB(CurrentTemp := CurrentQuenchTemp, 
                 TempSetpoint := QuenchTempSetpoint, 
                 TempTolerance := QuenchTempTolerance);
        QuenchCoolerOn := QuenchFB.CoolerCmd;
        
        (* Transition to drying *)
        IF QuenchFB.QuenchComplete AND UnitReady[4] AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransferPump[3] := TRUE; (* Quench to dryer *)
            TransitionDelay(IN := FALSE);
        ELSIF QuenchFB.QuenchComplete AND UnitReady[4] THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Drying *)
        (* Dry at 80°C for 3 hours *)
        DryFB(TempSetpoint := DryTempSetpoint, 
              CurrentTemp := CurrentDryTemp, 
              TempTolerance := DryTempTolerance, 
              Timer := DryTimer, 
              Duration := DryDuration);
        DryHeaterOn := DryFB.HeaterCmd;
        NitrogenValve := TRUE; (* Nitrogen for drying *)
        
        (* Transition to pelletizing *)
        IF DryFB.DryingComplete AND UnitReady[5] AND TransitionDelay.Q THEN
            PhaseState := 5;
            TransferPump[4] := TRUE; (* Dryer to pelletizer *)
            TransitionDelay(IN := FALSE);
        ELSIF DryFB.DryingComplete AND UnitReady[5] THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    5: (* Pelletizing *)
        (* Pelletize at 150°C, 100 RPM *)
        PelletizeFB(TempSetpoint := PelletTempSetpoint, 
                    CurrentTemp := CurrentPelletTemp, 
                    TempTolerance := PelletTempTolerance, 
                    SpeedSetpoint := PelletSpeedSetpoint, 
                    CurrentSpeed := CurrentPelletSpeed, 
                    SpeedTolerance := PelletSpeedTolerance);
        PelletHeaterOn := PelletizeFB.HeaterCmd;
        PelletMotorOn := PelletizeFB.MotorCmd;
        
        (* Transition to quality control *)
        IF PelletizeFB.PelletizingComplete AND UnitReady[6] AND TransitionDelay.Q THEN
            PhaseState := 6;
            TransferPump[5] := TRUE; (* Pelletizer to QC *)
            TransitionDelay(IN := FALSE);
        ELSIF PelletizeFB.PelletizingComplete AND UnitReady[6] THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    6: (* Quality Control *)
        (* Check melt index and density *)
        QualityFB(MeltIndex := MeltIndex, 
                  MeltIndexMax := MeltIndexMax, 
                  Density := Density, 
                  DensityMin := DensityMin, 
                  DensityMax := DensityMax);
        
        (* Transition to packaging *)
        IF QualityFB.QualityPassed AND UnitReady[7] AND TransitionDelay.Q THEN
            PhaseState := 7;
            TransferPump[6] := TRUE; (* QC to packaging *)
            TransitionDelay(IN := FALSE);
        ELSIF QualityFB.QualityPassed AND UnitReady[7] THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        ELSIF NOT QualityFB.QualityPassed THEN
            FaultState := TRUE;
            PhaseState := 9;
        END_IF;
    
    7: (* Packaging and Storage *)
        (* Package pellets *)
        PackageFB(QualityPassed := QualityFB.QualityPassed);
        PackageConveyor := PackageFB.ConveyorCmd;
        
        (* Transition to complete *)
        IF PackageFB.PackagingComplete AND TransitionDelay.Q THEN
            PhaseState := 8;
            BatchComplete := TRUE;
            TransitionDelay(IN := FALSE);
        ELSIF PackageFB.PackagingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    8: (* Complete State *)
        (* Reset outputs *)
        EthyleneValve := FALSE;
        CatalystValve := FALSE;
        SolventValve := FALSE;
        NitrogenValve := FALSE;
        PolyHeaterOn := FALSE;
        PolyCoolerOn := FALSE;
        PolyMixerOn := FALSE;
        QuenchCoolerOn := FALSE;
        DryHeaterOn := FALSE;
        PelletHeaterOn := FALSE;
        PelletMotorOn := FALSE;
        FOR i := 1 TO 6 DO
            TransferPump[i] := FALSE;
        END_FOR;
        PackageConveyor := FALSE;
        PhaseTimer(IN := FALSE);
        PolyTimer(IN := FALSE);
        DryTimer(IN := FALSE);
        
        (* Wait for reset *)
        IF ResetCommand THEN
            PhaseState := 0;
            BatchComplete := FALSE;
        END_IF;
    
    9: (* Fault State *)
        (* Disable all outputs *)
        EthyleneValve := FALSE;
        CatalystValve := FALSE;
        SolventValve := FALSE;
        NitrogenValve := FALSE;
        PolyHeaterOn := FALSE;
        PolyCoolerOn := FALSE;
        PolyMixerOn := FALSE;
        QuenchCoolerOn := FALSE;
        DryHeaterOn := FALSE;
        PelletHeaterOn := FALSE;
        PelletMotorOn := FALSE;
        FOR i := 1 TO 6 DO
            TransferPump[i] := FALSE;
        END_FOR;
        PackageConveyor := FALSE;
        PhaseTimer(IN := FALSE);
        PolyTimer(IN := FALSE);
        DryTimer(IN := FALSE);
        
        (* Require manual reset *)
        IF ResetCommand AND NOT EmergencyStop THEN
            PhaseState := 0;
            FaultState := FALSE;
        END_IF;
END_CASE;

(* Safety and Fault Monitoring *)
IF EmergencyStop THEN
    PhaseState := 9;
    FaultState := TRUE;
END_IF;

(* Polymerization Monitoring *)
IF (CurrentPolyTemp > (PolyTempSetpoint + 10.0) OR 
    CurrentPolyTemp < (PolyTempSetpoint - 10.0) OR
    CurrentPolyPressure > (PolyPressureSetpoint + 100.0) OR
    CurrentPolySpeed > (PolyMixingSpeed + 50.0)) AND PhaseState = 2 THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

(* Quenching Monitoring *)
IF CurrentQuenchTemp > (QuenchTempSetpoint + 10.0) AND PhaseState = 3 THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

(* Drying Monitoring *)
IF CurrentDryTemp > (DryTempSetpoint + 5.0) AND PhaseState = 4 THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

(* Pelletizing Monitoring *)
IF (CurrentPelletTemp > (PelletTempSetpoint + 10.0) OR
    CurrentPelletSpeed > (PelletSpeedSetpoint + 20.0)) AND PhaseState = 5 THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

(* Resource Contention Interlock *)
(* Prioritize reactor pressure control *)
IF PhaseState = 2 AND CurrentPolyPressure > PolyPressureSetpoint THEN
    TransferPump[1] := FALSE; (* Block prep transfer during high pressure *)
END_IF;

(* Synchronization Interlock *)
FOR i := 2 TO 7 DO
    IF PhaseState = i-1 AND NOT UnitReady[i] THEN
        (* Pause if next unit is not ready *)
        PhaseTimer(IN := FALSE);
    ELSE
        PhaseTimer(IN := TRUE);
    END_IF;
END_FOR;

END_PROGRAM
