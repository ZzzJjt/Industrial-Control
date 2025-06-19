(* IEC 61131-3 Structured Text: ISA-88 Polyethylene Batch Control *)
(* Purpose: Controls entire polyethylene production cycle with modular phases *)

(* Function Block: Dose Material *)
FUNCTION_BLOCK DoseMaterial
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Mass_SP : REAL;                 (* Target mass in kg *)
    Mass_Tol : REAL;                (* Mass tolerance in kg *)
    Mass_PV : REAL;                 (* Measured mass in kg *)
END_VAR
VAR_OUTPUT
    ValveOn : BOOL;                 (* TRUE to open dosing valve *)
    Complete : BOOL;                (* TRUE when dosing complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Dosing, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    ValveOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            ValveOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Dosing *)
            ValveOn := TRUE;
            IF ABS(Mass_PV - Mass_SP) <= Mass_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            ValveOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Ensures precise material dosing for consistent batch composition *)
END_FUNCTION_BLOCK

(* Function Block: Dose Solvent *)
FUNCTION_BLOCK DoseSolvent
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Volume_SP : REAL;               (* Target volume in liters *)
    Volume_Tol : REAL;              (* Volume tolerance in liters *)
    Volume_PV : REAL;               (* Measured volume in liters *)
END_VAR
VAR_OUTPUT
    ValveOn : BOOL;                 (* TRUE to open solvent valve *)
    Complete : BOOL;                (* TRUE when dosing complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Dosing, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    ValveOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            ValveOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Dosing *)
            ValveOn := TRUE;
            IF ABS(Volume_PV - Volume_SP) <= Volume_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            ValveOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Doses solvent accurately to support polymerization *)
END_FUNCTION_BLOCK

(* Function Block: Pressurize Reactor *)
FUNCTION_BLOCK PressurizeReactor
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Pressure_SP : REAL;             (* Target pressure in bar *)
    Pressure_Tol : REAL;            (* Pressure tolerance in bar *)
    Pressure_PV : REAL;             (* Measured pressure in bar *)
END_VAR
VAR_OUTPUT
    CompressorOn : BOOL;            (* TRUE to activate compressor *)
    Complete : BOOL;                (* TRUE when pressure reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Pressurizing, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    CompressorOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            CompressorOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Pressurizing *)
            CompressorOn := TRUE;
            IF ABS(Pressure_PV - Pressure_SP) <= Pressure_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            CompressorOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Achieves high pressure for polymerization reaction *)
END_FUNCTION_BLOCK

(* Function Block: Heat Operation *)
FUNCTION_BLOCK HeatOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    Complete : BOOL;                (* TRUE when temperature reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Heating, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Heating *)
            HeaterOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Ensures precise temperature for reaction or drying *)
END_FUNCTION_BLOCK

(* Function Block: Polymerization *)
FUNCTION_BLOCK PolymerizeOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Duration : TIME;                (* Reaction duration *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    AgitatorOn : BOOL;              (* TRUE to activate agitator *)
    Complete : BOOL;                (* TRUE when reaction complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Reacting, 2=Complete *)
    PolyTimer : TON;                (* Timer for reaction *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    AgitatorOn := FALSE;
    Complete := FALSE;
    PolyTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            AgitatorOn := FALSE;
            Complete := FALSE;
            State := 1;
            PolyTimer(IN := TRUE, PT := Duration);
        1: (* Reacting *)
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol;
            AgitatorOn := TRUE;
            IF PolyTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            AgitatorOn := FALSE;
            Complete := TRUE;
            PolyTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Maintains reaction conditions for polyethylene formation *)
END_FUNCTION_BLOCK

(* Function Block: Quench Operation *)
FUNCTION_BLOCK QuenchOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Pressure_SP : REAL;             (* Target pressure in bar *)
    Pressure_Tol : REAL;            (* Pressure tolerance in bar *)
    Pressure_PV : REAL;             (* Measured pressure in bar *)
END_VAR
VAR_OUTPUT
    CoolerOn : BOOL;                (* TRUE to activate cooler *)
    VentValveOn : BOOL;             (* TRUE to open vent valve *)
    Complete : BOOL;                (* TRUE when quenching complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Quenching, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    CoolerOn := FALSE;
    VentValveOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            CoolerOn := FALSE;
            VentValveOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Quenching *)
            CoolerOn := TRUE;
            VentValveOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol AND ABS(Pressure_PV - Pressure_SP) <= Pressure_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            CoolerOn := FALSE;
            VentValveOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Rapidly cools and depressurizes reactor to stop reaction *)
END_FUNCTION_BLOCK

(* Function Block: Dry Operation *)
FUNCTION_BLOCK DryOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Duration : TIME;                (* Drying duration *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    Complete : BOOL;                (* TRUE when drying complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Heating, 2=Drying, 3=Complete *)
    DryTimer : TON;                 (* Timer for drying *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    Complete := FALSE;
    DryTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Heating *)
            HeaterOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
                DryTimer(IN := TRUE, PT := Duration);
            END_IF;
        2: (* Drying *)
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol;
            IF DryTimer.Q THEN
                State := 3;
            END_IF;
        3: (* Complete *)
            HeaterOn := FALSE;
            Complete := TRUE;
            DryTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Removes residual solvent for pure polymer *)
END_FUNCTION_BLOCK

(* Function Block: Pelletize Operation *)
FUNCTION_BLOCK PelletizeOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    ExtruderOn : BOOL;              (* TRUE to activate extruder *)
    Complete : BOOL;                (* TRUE when pelletizing complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Heating, 2=Pelletizing, 3=Complete *)
    PelletTimer : TON;              (* Timer for pelletizing *)
    PelletDuration : TIME := T#30m; (* Fixed pelletizing duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    ExtruderOn := FALSE;
    Complete := FALSE;
    PelletTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            ExtruderOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Heating *)
            ExtruderOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
                PelletTimer(IN := TRUE, PT := PelletDuration);
            END_IF;
        2: (* Pelletizing *)
            ExtruderOn := TRUE;
            IF PelletTimer.Q THEN
                State := 3;
            END_IF;
        3: (* Complete *)
            ExtruderOn := FALSE;
            Complete := TRUE;
            PelletTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Forms uniform polyethylene pellets *)
END_FUNCTION_BLOCK

(* Function Block: Quality Control *)
FUNCTION_BLOCK QualityControl
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Density_PV : REAL;              (* Measured density in g/cm³ *)
    Density_SP : REAL;              (* Target density *)
    Density_Tol : REAL;             (* Density tolerance *)
    MeltIndex_PV : REAL;            (* Measured melt index *)
    MeltIndex_SP : REAL;            (* Target melt index *)
    MeltIndex_Tol : REAL;           (* Melt index tolerance *)
END_VAR
VAR_OUTPUT
    TestActive : BOOL;              (* TRUE during testing *)
    Pass : BOOL;                    (* TRUE if quality passes *)
    Complete : BOOL;                (* TRUE when testing complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Testing, 2=Complete *)
    TestTimer : TON;                (* Timer for testing *)
    TestDuration : TIME := T#10m;   (* Fixed test duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    TestActive := FALSE;
    Pass := FALSE;
    Complete := FALSE;
    TestTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            TestActive := FALSE;
            Pass := FALSE;
            Complete := FALSE;
            State := 1;
            TestTimer(IN := TRUE, PT := TestDuration);
        1: (* Testing *)
            TestActive := TRUE;
            Pass := (ABS(Density_PV - Density_SP) <= Density_Tol) AND 
                    (ABS(MeltIndex_PV - MeltIndex_SP) <= MeltIndex_Tol);
            IF TestTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            TestActive := FALSE;
            Complete := TRUE;
            TestTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Verifies pellet quality before packaging *)
END_FUNCTION_BLOCK

(* Function Block: Packaging *)
FUNCTION_BLOCK PackagingOperation
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    BagMass_SP : REAL;              (* Target bag mass in kg *)
    BagMass_Tol : REAL;             (* Bag mass tolerance in kg *)
    BagMass_PV : REAL;              (* Measured bag mass in kg *)
END_VAR
VAR_OUTPUT
    BaggerOn : BOOL;                (* TRUE to activate bagger *)
    ConveyorOn : BOOL;              (* TRUE to activate conveyor *)
    Complete : BOOL;                (* TRUE when packaging complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Packaging, 2=Complete *)
    BagCount : INT := 0;            (* Number of bags filled *)
    MaxBags : INT := 20;            (* 1000 kg / 25 kg per bag ≈ 20 bags *)
END_VAR
IF NOT Enable THEN
    State := 0;
    BaggerOn := FALSE;
    ConveyorOn := FALSE;
    Complete := FALSE;
    BagCount := 0;
ELSE
    CASE State OF
        0: (* Idle *)
            BaggerOn := FALSE;
            ConveyorOn := FALSE;
            Complete := FALSE;
            BagCount := 0;
            State := 1;
        1: (* Packaging *)
            BaggerOn := TRUE;
            ConveyorOn := TRUE;
            IF ABS(BagMass_PV - BagMass_SP) <= BagMass_Tol THEN
                BagCount := BagCount + 1;
                IF BagCount >= MaxBags THEN
                    State := 2;
                END_IF;
            END_IF;
        2: (* Complete *)
            BaggerOn := FALSE;
            ConveyorOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Packages pellets into 25 kg bags for storage *)
END_FUNCTION_BLOCK

(* Main Program: Polyethylene Batch Control *)
PROGRAM PolyethyleneBatchControl
VAR
    (* Inputs *)
    StartBatch : BOOL;              (* TRUE to start batch *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    EthyleneMass_PV : REAL;         (* Measured ethylene mass in kg *)
    CatalystMass_PV : REAL;         (* Measured catalyst mass in kg *)
    SolventVolume_PV : REAL;        (* Measured solvent volume in liters *)
    ReactorPressure_PV : REAL;      (* Reactor pressure in bar *)
    ReactorTemp_PV : REAL;          (* Reactor temperature in °C *)
    DryerTemp_PV : REAL;            (* Dryer temperature in °C *)
    ExtruderTemp_PV : REAL;         (* Extruder temperature in °C *)
    Density_PV : REAL;              (* Measured pellet density in g/cm³ *)
    MeltIndex_PV : REAL;            (* Measured melt index *)
    BagMass_PV : REAL;              (* Measured bag mass in kg *)
    
    (* Outputs *)
    EthyleneValveOn : BOOL;         (* TRUE to dose ethylene *)
    CatalystValveOn : BOOL;         (* TRUE to dose catalyst *)
    SolventValveOn : BOOL;          (* TRUE to dose solvent *)
    CompressorOn : BOOL;            (* TRUE to activate compressor *)
    ReactorHeaterOn : BOOL;         (* TRUE to activate reactor heater *)
    ReactorAgitatorOn : BOOL;       (* TRUE to activate agitator *)
    ReactorCoolerOn : BOOL;         (* TRUE to activate cooler *)
    VentValveOn : BOOL;             (* TRUE to open vent valve *)
    DryerHeaterOn : BOOL;           (* TRUE to activate dryer heater *)
    ExtruderOn : BOOL;              (* TRUE to activate extruder *)
    QCTestActive : BOOL;            (* TRUE during quality testing *)
    QCPass : BOOL;                  (* TRUE if quality passes *)
    BaggerOn : BOOL;                (* TRUE to activate bagger *)
    ConveyorOn : BOOL;              (* TRUE to activate conveyor *)
    BatchComplete : BOOL;           (* TRUE when batch complete *)
    
    (* State Machine *)
    State : INT := 0;               (* Main state *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_PREPARATION : INT := 1; (* Raw material preparation *)
        STATE_POLYMERIZATION : INT := 2; (* Polymerization *)
        STATE_QUENCHING : INT := 3; (* Quenching *)
        STATE_DRYING : INT := 4;    (* Drying *)
        STATE_PELLETIZING : INT := 5; (* Pelletizing *)
        STATE_QUALITY_CONTROL : INT := 6; (* Quality control *)
        STATE_PACKAGING : INT := 7; (* Packaging and storage *)
        STATE_COMPLETE : INT := 8;  (* Batch complete *)
    END_CONSTANT
    
    (* Sub-State for Preparation *)
    PrepSubState : INT := 0;        (* 0=Idle, 1=DoseEthylene, 2=DoseCatalyst, 3=DoseSolvent *)
    CONSTANT
        PREP_IDLE : INT := 0;
        PREP_DOSE_ETHYLENE : INT := 1;
        PREP_DOSE_CATALYST : INT := 2;
        PREP_DOSE_SOLVENT : INT := 3;
        PREP_COMPLETE : INT := 4;
    END_CONSTANT
    
    (* Sub-State for Polymerization *)
    PolySubState : INT := 0;        (* 0=Idle, 1=Pressurize, 2=Heat, 3=React *)
    CONSTANT
        POLY_IDLE : INT := 0;
        POLY_PRESSURIZE : INT := 1;
        POLY_HEAT : INT := 2;
        POLY_REACT : INT := 3;
        POLY_COMPLETE : INT := 4;
    END_CONSTANT
    
    (* Recipe Parameters *)
    EthyleneMass_SP : REAL := 500.0; (* Ethylene: 500 kg *)
    CatalystMass_SP : REAL := 0.5;  (* Catalyst: 0.5 kg *)
    SolventVolume_SP : REAL := 1000.0; (* Solvent: 1000 L *)
    Mass_Tol : REAL := 0.5;         (* Mass tolerance: ±0.5 kg *)
    Volume_Tol : REAL := 10.0;      (* Volume tolerance: ±10 L *)
    ReactorPressure_SP : REAL := 1000.0; (* Polymerization pressure: 1000 bar *)
    Pressure_Tol : REAL := 50.0;    (* Pressure tolerance: ±50 bar *)
    PolymerizationTemp_SP : REAL := 200.0; (* Polymerization temp: 200°C *)
    PolymerizationTemp_Tol : REAL := 5.0; (* Temp tolerance: ±5°C *)
    PolymerizationDuration : TIME := T#2h; (* Reaction duration: 2 hours *)
    QuenchTemp_SP : REAL := 50.0;   (* Quench temp: 50°C *)
    QuenchTemp_Tol : REAL := 3.0;   (* Quench temp tolerance: ±3°C *)
    QuenchPressure_SP : REAL := 1.0; (* Quench pressure: 1 bar *)
    DryTemp_SP : REAL := 80.0;      (* Dry temp: 80°C *)
    DryTemp_Tol : REAL := 3.0;      (* Dry temp tolerance: ±3°C *)
    DryDuration : TIME := T#3h;     (* Dry duration: 3 hours *)
    PelletTemp_SP : REAL := 180.0;  (* Pelletizing temp: 180°C *)
    PelletTemp_Tol : REAL := 5.0;   (* Pelletizing temp tolerance: ±5°C *)
    Density_SP : REAL := 0.95;      (* Target density: 0.95 g/cm³ *)
    Density_Tol : REAL := 0.02;     (* Density tolerance: ±0.02 g/cm³ *)
    MeltIndex_SP : REAL := 2.0;     (* Target melt index *)
    MeltIndex_Tol : REAL := 0.2;    (* Melt index tolerance: ±0.2 *)
    BagMass_SP : REAL := 25.0;      (* Bag mass: 25 kg *)
    BagMass_Tol : REAL := 0.5;      (* Bag mass tolerance: ±0.5 kg *)
    
    (* Operation Instances *)
    DoseEthyleneOp : DoseMaterial;  (* Dose ethylene *)
    DoseCatalystOp : DoseMaterial;  (* Dose catalyst *)
    DoseSolventOp : DoseSolvent;    (* Dose solvent *)
    PressurizeOp : PressurizeReactor; (* Pressurize reactor *)
    HeatOp : HeatOperation;         (* Heat operation *)
    PolyOp : PolymerizeOperation;   (* Polymerization *)
    QuenchOp : QuenchOperation;     (* Quenching *)
    DryOp : DryOperation;           (* Drying *)
    PelletOp : PelletizeOperation;  (* Pelletizing *)
    QCOp : QualityControl;          (* Quality control *)
    PackOp : PackagingOperation;    (* Packaging *)
    
    (* Resource Management *)
    CoolingSystemAvailable : BOOL := TRUE; (* TRUE if cooling system free *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    PrepSubState := PREP_IDLE;
    PolySubState := POLY_IDLE;
    EthyleneValveOn := FALSE;
    CatalystValveOn := FALSE;
    SolventValveOn := FALSE;
    CompressorOn := FALSE;
    ReactorHeaterOn := FALSE;
    ReactorAgitatorOn := FALSE;
    ReactorCoolerOn := FALSE;
    VentValveOn := FALSE;
    DryerHeaterOn := FALSE;
    ExtruderOn := FALSE;
    QCTestActive := FALSE;
    QCPass := FALSE;
    BaggerOn := FALSE;
    ConveyorOn := FALSE;
    BatchComplete := FALSE;
    CoolingSystemAvailable := TRUE;
    DoseEthyleneOp.Enable := FALSE;
    DoseCatalystOp.Enable := FALSE;
    DoseSolventOp.Enable := FALSE;
    PressurizeOp.Enable := FALSE;
    HeatOp.Enable := FALSE;
    PolyOp.Enable := FALSE;
    QuenchOp.Enable := FALSE;
    DryOp.Enable := FALSE;
    PelletOp.Enable := FALSE;
    QCOp.Enable := FALSE;
    PackOp.Enable := FALSE;
ELSIF StartBatch AND State = STATE_IDLE THEN
    State := STATE_PREPARATION;
    PrepSubState := PREP_DOSE_ETHYLENE;
    BatchComplete := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        EthyleneValveOn := FALSE;
        CatalystValveOn := FALSE;
        SolventValveOn := FALSE;
        CompressorOn := FALSE;
        ReactorHeaterOn := FALSE;
        ReactorAgitatorOn := FALSE;
        ReactorCoolerOn := FALSE;
        VentValveOn := FALSE;
        DryerHeaterOn := FALSE;
        ExtruderOn := FALSE;
        QCTestActive := FALSE;
        QCPass := FALSE;
        BaggerOn := FALSE;
        ConveyorOn := FALSE;
        BatchComplete := FALSE;
        PrepSubState := PREP_IDLE;
        PolySubState := POLY_IDLE;
        CoolingSystemAvailable := TRUE;
        DoseEthyleneOp.Enable := FALSE;
        DoseCatalystOp.Enable := FALSE;
        DoseSolventOp.Enable := FALSE;
        PressurizeOp.Enable := FALSE;
        HeatOp.Enable := FALSE;
        PolyOp.Enable := FALSE;
        QuenchOp.Enable := FALSE;
        DryOp.Enable := FALSE;
        PelletOp.Enable := FALSE;
        QCOp.Enable := FALSE;
        PackOp.Enable := FALSE;
        (* Idle state: All equipment off, ready for new batch *)

    STATE_PREPARATION:
        CASE PrepSubState OF
            PREP_IDLE:
                PrepSubState := PREP_DOSE_ETHYLENE;
            PREP_DOSE_ETHYLENE:
                DoseEthyleneOp(Enable := TRUE, Mass_SP := EthyleneMass_SP, Mass_Tol := Mass_Tol, Mass_PV := EthyleneMass_PV);
                EthyleneValveOn := DoseEthyleneOp.ValveOn;
                IF DoseEthyleneOp.Complete THEN
                    PrepSubState := PREP_DOSE_CATALYST;
                    DoseEthyleneOp.Enable := FALSE;
                END_IF;
                (* Doses ethylene for polymerization *)
            PREP_DOSE_CATALYST:
                DoseCatalystOp(Enable := TRUE, Mass_SP := CatalystMass_SP, Mass_Tol := Mass_Tol, Mass_PV := CatalystMass_PV);
                CatalystValveOn := DoseCatalystOp.ValveOn;
                IF DoseCatalystOp.Complete THEN
                    PrepSubState := PREP_DOSE_SOLVENT;
                    DoseCatalystOp.Enable := FALSE;
                END_IF;
                (* Doses catalyst to initiate reaction *)
            PREP_DOSE_SOLVENT:
                DoseSolventOp(Enable := TRUE, Volume_SP := SolventVolume_SP, Volume_Tol := Volume_Tol, Volume_PV := SolventVolume_PV);
                SolventValveOn := DoseSolventOp.ValveOn;
                IF DoseSolventOp.Complete THEN
                    PrepSubState := PREP_COMPLETE;
                    DoseSolventOp.Enable := FALSE;
                END_IF;
                (* Doses solvent to support reaction *)
            PREP_COMPLETE:
                State := STATE_POLYMERIZATION;
                PolySubState := POLY_PRESSURIZE;
        END_CASE;

    STATE_POLYMERIZATION:
        CASE PolySubState OF
            POLY_IDLE:
                PolySubState := POLY_PRESSURIZE;
            POLY_PRESSURIZE:
                PressurizeOp(Enable := TRUE, Pressure_SP := ReactorPressure_SP, Pressure_Tol := Pressure_Tol, Pressure_PV := ReactorPressure_PV);
                CompressorOn := PressurizeOp.CompressorOn;
                IF PressurizeOp.Complete THEN
                    PolySubState := POLY_HEAT;
                    PressurizeOp.Enable := FALSE;
                END_IF;
                (* Pressurizes reactor for high-pressure polymerization *)
            POLY_HEAT:
                HeatOp(Enable := TRUE, Temp_SP := PolymerizationTemp_SP, Temp_Tol := PolymerizationTemp_Tol, Temp_PV := ReactorTemp_PV);
                ReactorHeaterOn := HeatOp.HeaterOn;
                IF HeatOp.Complete THEN
                    PolySubState := POLY_REACT;
                    HeatOp.Enable := FALSE;
                END_IF;
                (* Heats reactor to reaction temperature *)
            POLY_REACT:
                PolyOp(Enable := TRUE, Temp_SP := PolymerizationTemp_SP, Temp_Tol := PolymerizationTemp_Tol, Temp_PV := ReactorTemp_PV, Duration := PolymerizationDuration);
                ReactorHeaterOn := PolyOp.HeaterOn;
                ReactorAgitatorOn := PolyOp.AgitatorOn;
                IF PolyOp.Complete THEN
                    PolySubState := POLY_COMPLETE;
                    PolyOp.Enable := FALSE;
                END_IF;
                (* Maintains reaction conditions for polyethylene formation *)
            POLY_COMPLETE:
                ReactorHeaterOn := FALSE;
                ReactorAgitatorOn := FALSE;
                IF CoolingSystemAvailable THEN
                    State := STATE_QUENCHING;
                    CoolingSystemAvailable := FALSE;
                END_IF;
                (* Transitions to quenching when cooling system is available *)
        END_CASE;

    STATE_QUENCHING:
        QuenchOp(Enable := TRUE, Temp_SP := QuenchTemp_SP, Temp_Tol := QuenchTemp_Tol, Temp_PV := ReactorTemp_PV, Pressure_SP := QuenchPressure_SP, Pressure_Tol := Pressure_Tol, Pressure_PV := ReactorPressure_PV);
        ReactorCoolerOn := QuenchOp.CoolerOn;
        VentValveOn := QuenchOp.VentValveOn;
        IF QuenchOp.Complete THEN
            State := STATE_DRYING;
            QuenchOp.Enable := FALSE;
            CoolingSystemAvailable := TRUE;
        END_IF;
        (* Cools and depressurizes reactor to stop reaction *)

    STATE_DRYING:
        DryOp(Enable := TRUE, Temp_SP := DryTemp_SP, Temp_Tol := DryTemp_Tol, Temp_PV := DryerTemp_PV, Duration := DryDuration);
        DryerHeaterOn := DryOp.HeaterOn;
        IF DryOp.Complete THEN
            State := STATE_PELLETIZING;
            DryOp.Enable := FALSE;
        END_IF;
        (* Dries polymer to remove solvent *)

    STATE_PELLETIZING:
        PelletOp(Enable := TRUE, Temp_SP := PelletTemp_SP, Temp_Tol := PelletTemp_Tol, Temp_PV := ExtruderTemp_PV);
        ExtruderOn := PelletOp.ExtruderOn;
        IF PelletOp.Complete THEN
            State := STATE_QUALITY_CONTROL;
            PelletOp.Enable := FALSE;
        END_IF;
        (* Forms uniform polyethylene pellets *)

    STATE_QUALITY_CONTROL:
        QCOp(Enable := TRUE, Density_PV := Density_PV, Density_SP := Density_SP, Density_Tol := Density_Tol, MeltIndex_PV := MeltIndex_PV, MeltIndex_SP := MeltIndex_SP, MeltIndex_Tol := MeltIndex_Tol);
        QCTestActive := QCOp.TestActive;
        QCPass := QCOp.Pass;
        IF QCOp.Complete AND QCOp.Pass THEN
            State := STATE_PACKAGING;
            QCOp.Enable := FALSE;
        ELSIF QCOp.Complete AND NOT QCOp.Pass THEN
            State := STATE_IDLE; (* Reject batch if quality fails *)
            QCOp.Enable := FALSE;
        END_IF;
        (* Verifies pellet quality; proceeds only if pass *)

    STATE_PACKAGING:
        PackOp(Enable := TRUE, BagMass_SP := BagMass_SP, BagMass_Tol := BagMass_Tol, BagMass_PV := BagMass_PV);
        BaggerOn := PackOp.BaggerOn;
        ConveyorOn := PackOp.ConveyorOn;
        IF PackOp.Complete THEN
            State := STATE_COMPLETE;
            PackOp.Enable := FALSE;
        END_IF;
        (* Packages pellets into 25 kg bags *)

    STATE_COMPLETE:
        EthyleneValveOn := FALSE;
        CatalystValveOn := FALSE;
        SolventValveOn := FALSE;
        CompressorOn := FALSE;
        ReactorHeaterOn := FALSE;
        ReactorAgitatorOn := FALSE;
        ReactorCoolerOn := FALSE;
        VentValveOn := FALSE;
        DryerHeaterOn := FALSE;
        ExtruderOn := FALSE;
        QCTestActive := FALSE;
        BaggerOn := FALSE;
        ConveyorOn := FALSE;
        BatchComplete := TRUE;
        IF NOT StartBatch THEN
            State := STATE_IDLE;
        END_IF;
        (* Batch complete; reset for next batch *)

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Unit Procedures: Preparation, Polymerization, Quenching, Drying, Pelletizing, Quality Control, Packaging.
     - Operations: Dose, Pressurize, Heat, Polymerize, Quench, Dry, Pelletize, QualityControl, Packaging as function blocks.
     - Recipe Parameters: Mass_SP, Temp_SP, Duration, etc., for flexibility.
     - State Model: Idle, Running, Complete per ISA-88.
   - Process Stages:
     - Preparation: Dose ethylene (500 kg), catalyst (0.5 kg), solvent (1000 L).
     - Polymerization: Pressurize to 1000 bar, heat to 200°C, react for 2 hours.
     - Quenching: Cool to 50°C, depressurize to 1 bar.
     - Drying: Dry at 80°C for 3 hours.
     - Pelletizing: Extrude at 180°C for 30 min.
     - Quality Control: Test density (0.95 g/cm³), melt index (2.0).
     - Packaging: Fill 25 kg bags (20 bags).
   - Safety:
     - EmergencyStop halts all operations, resets state.
     - Tolerances (0.5 kg, 10 L, 50 bar, 3–5°C) ensure quality.
   - Quality:
     - Precise dosing, temperature, and pressure ensure consistent polymer properties.
     - Quality control rejects substandard batches.
   - Resource Sharing:
     - CoolingSystemAvailable flag prevents conflicts between quenching and other cooling needs.
   - Physical Integration:
     - Inputs: StartBatch, Mass_PV, Temp_PV, Pressure_PV, etc. (sensors).
     - Outputs: ValveOn, CompressorOn, HeaterOn, etc. (relays).
   - Scalability:
     - Adjust parameters (Mass_SP, Duration) for larger batches.
     - Modular blocks support additional operations or units.
   - Maintenance:
     - Function blocks simplify debugging.
     - Monitor Temp_PV, Pressure_PV, QCOp.Pass on HMI.
*)
END_PROGRAM
