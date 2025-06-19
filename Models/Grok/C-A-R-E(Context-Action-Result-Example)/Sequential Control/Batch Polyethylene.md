(* IEC 61131-3 Structured Text Program: PolyethyleneBatchControl *)
(* Purpose: Implements ISA-88 batch control for 500 kg polyethylene production *)
(* Overview: Manages raw material preparation, polymerization, quenching, drying, pelletizing, quality control, and packaging with modular phases *)

PROGRAM PolyethyleneBatchControl
VAR
    (* Inputs *)
    StartBatch : R_TRIG;            (* Rising edge to start batch *)
    StopBatch : R_TRIG;             (* Rising edge to stop batch *)
    CurrentWeight : REAL;           (* Tank/reactor weight, kg *)
    CurrentTemp : REAL;             (* Reactor/dryer temperature, °C *)
    CurrentPressure : REAL;         (* Reactor pressure, bar *)
    CurrentAgitSpeed : REAL;        (* Agitator speed, RPM *)
    CurrentViscosity : REAL;        (* Product viscosity, cP *)
    EmergencyStop : BOOL;           (* TRUE if emergency stop triggered *)

    (* Outputs *)
    MonomerValve : BOOL;            (* TRUE to dose monomers *)
    CatalystValve : BOOL;           (* TRUE to dose catalysts *)
    SolventValve : BOOL;            (* TRUE to dose solvents *)
    CoolantValve : BOOL;            (* TRUE to enable coolant flow *)
    TargetTemp : REAL;              (* Temperature setpoint, °C *)
    TargetAgitSpeed : REAL;         (* Agitator speed setpoint, RPM *)
    HeaterControl : BOOL;           (* TRUE to activate heater *)
    CoolerControl : BOOL;           (* TRUE to activate cooler *)
    AgitatorControl : BOOL;         (* TRUE to activate agitator *)
    TransferConveyor : BOOL;        (* TRUE to transfer to dryer/pelletizer *)
    PelletizerControl : BOOL;       (* TRUE to activate pelletizer *)
    PackagingControl : BOOL;        (* TRUE to activate packaging *)
    BatchComplete : BOOL;           (* TRUE when batch is finished *)
    State : INT := 0;               (* Current state: 0=IDLE, 1=DOSE_MONOMERS, 2=DOSE_CATALYSTS, 3=DOSE_SOLVENTS, 4=POLY_HEAT_AGITATE, 5=POLY_HOLD, 6=QUENCH, 7=DRY_TRANSFER, 8=DRY_DRY, 9=PELLETIZE, 10=QUALITY_CHECK, 11=PACKAGE, 12=COMPLETED, 13=ERROR *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    PhaseTimer : TON;               (* Timer for phase durations *)
    InitialWeight : REAL;           (* Weight before dosing *)
    Last_StartBatch : BOOL;         (* Previous StartBatch.Q *)
    Last_StopBatch : BOOL;         (* Previous StopBatch.Q *)
    Last_BatchComplete : BOOL;      (* Previous BatchComplete *)
    Last_State : INT;               (* Previous State *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Parameter Constants *)
VAR CONSTANT
    (* Ingredient Quantities *)
    MONOMER_QTY : REAL := 300.0;    (* Monomers, kg *)
    CATALYST_QTY : REAL := 5.0;     (* Catalysts, kg *)
    SOLVENT_QTY : REAL := 150.0;    (* Solvents, kg *)
    (* Process Parameters *)
    POLY_TEMP : REAL := 80.0;       (* Polymerization temperature, °C *)
    POLY_TEMP_TOL : REAL := 2.0;    (* ±2°C tolerance *)
    POLY_SPEED : REAL := 120.0;     (* Agitation speed, RPM *)
    POLY_SPEED_TOL : REAL := 10.0;  (* ±10 RPM tolerance *)
    POLY_PRESS : REAL := 2.0;       (* Pressure threshold, bar *)
    QUENCH_TEMP : REAL := 30.0;     (* Quenching temperature, °C *)
    QUENCH_TEMP_TOL : REAL := 2.0;  (* ±2°C tolerance *)
    DRY_TEMP : REAL := 90.0;        (* Drying temperature, °C *)
    DRY_TEMP_TOL : REAL := 2.0;     (* ±2°C tolerance *)
    DRY_SPEED : REAL := 100.0;      (* Drying agitation speed, RPM *)
    DRY_SPEED_TOL : REAL := 10.0;   (* ±10 RPM tolerance *)
    PELLET_SPEED : REAL := 200.0;   (* Pelletizing speed, RPM *)
    PELLET_SPEED_TOL : REAL := 20.0; (* ±20 RPM tolerance *)
    VISCOSITY_TARGET : REAL := 500.0; (* Quality viscosity, cP *)
    VISCOSITY_TOL : REAL := 50.0;   (* ±50 cP tolerance *)
    (* Durations *)
    DOSE_TIMEOUT : TIME := T#10m;   (* Dosing timeout *)
    HEAT_AGITATE_TIMEOUT : TIME := T#30m; (* Heat/agitate timeout *)
    HOLD_MAX_TIME : TIME := T#2h;   (* Max reaction hold time *)
    QUENCH_TIMEOUT : TIME := T#20m; (* Quenching timeout *)
    TRANSFER_TIMEOUT : TIME := T#10m; (* Transfer timeout *)
    DRY_DURATION : TIME := T#1h;    (* Drying duration *)
    PELLET_DURATION : TIME := T#30m; (* Pelletizing duration *)
    QUALITY_DURATION : TIME := T#5m; (* Quality check duration *)
    PACKAGE_DURATION : TIME := T#10m; (* Packaging duration *)
    (* Input Validation Ranges *)
    MIN_WEIGHT : REAL := 0.0;       (* Min valid weight, kg *)
    MAX_WEIGHT : REAL := 1000.0;    (* Max valid weight, kg *)
    MIN_TEMP : REAL := 0.0;         (* Min valid temperature, °C *)
    MAX_TEMP : REAL := 200.0;       (* Max valid temperature, °C *)
    MIN_PRESS : REAL := 0.0;        (* Min valid pressure, bar *)
    MAX_PRESS : REAL := 10.0;       (* Max valid pressure, bar *)
    MIN_SPEED : REAL := 0.0;        (* Min valid speed, RPM *)
    MAX_SPEED : REAL := 500.0;      (* Max valid speed, RPM *)
    MIN_VISCOSITY : REAL := 0.0;    (* Min valid viscosity, cP *)
    MAX_VISCOSITY : REAL := 1000.0; (* Max valid viscosity, cP *)
END_VAR

(* Method: DoseIngredient *)
(* Opens valve, doses specified quantity, monitors weight *)
METHOD PRIVATE DoseIngredient : BOOL
VAR_INPUT
    Valve : BOOL;                   (* Valve to open *)
    TargetQty : REAL;               (* Target quantity, kg *)
END_VAR
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    Valve := TRUE;                  (* Activate specified valve *)
    DoseIngredient := CurrentWeight >= InitialWeight + TargetQty; (* TRUE if qty reached *)
    IF LogCount < 50 AND (MonomerValve <> Last_MonomerValve OR CatalystValve <> Last_CatalystValve OR 
                          SolventValve <> Last_SolventValve) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Dosing Ingredient: TargetQty=', 
            CONCAT(TO_STRING(TargetQty), ' kg')));
    END_IF;
END_METHOD

(* Method: HeatToTemp *)
(* Sets target temperature, activates heater, checks stability *)
METHOD PRIVATE HeatToTemp : BOOL
VAR_INPUT
    SetTemp : REAL;                 (* Target temperature, °C *)
END_VAR
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := TRUE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := SetTemp;
    TargetAgitSpeed := 0.0;
    HeatToTemp := ABS(CurrentTemp - SetTemp) <= POLY_TEMP_TOL; (* TRUE if temp stable *)
    IF LogCount < 50 AND (TargetTemp <> Last_TargetTemp OR HeaterControl <> Last_HeaterControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Heating Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), ' °C')));
    END_IF;
END_METHOD

(* Method: StartAgitation *)
(* Sets agitation speed, activates agitator *)
METHOD PRIVATE StartAgitation : BOOL
VAR_INPUT
    SetSpeed : REAL;                (* Target speed, RPM *)
END_VAR
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := TRUE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := SetSpeed;
    StartAgitation := ABS(CurrentAgitSpeed - SetSpeed) <= POLY_SPEED_TOL; (* TRUE if speed stable *)
    IF LogCount < 50 AND (TargetAgitSpeed <> Last_TargetAgitSpeed OR AgitatorControl <> Last_AgitatorControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Agitation Started: TargetSpeed=', 
            CONCAT(TO_STRING(TargetAgitSpeed), ' RPM')));
    END_IF;
END_METHOD

(* Method: HoldReaction *)
(* Maintains reaction conditions, checks pressure *)
METHOD PRIVATE HoldReaction : BOOL
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := TRUE;
    CoolerControl := FALSE;
    AgitatorControl := TRUE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := POLY_TEMP;        (* 80°C *)
    TargetAgitSpeed := POLY_SPEED;  (* 120 RPM *)
    HoldReaction := CurrentTemp >= POLY_TEMP - POLY_TEMP_TOL AND 
                    CurrentTemp <= POLY_TEMP + POLY_TEMP_TOL AND 
                    ABS(CurrentAgitSpeed - POLY_SPEED) <= POLY_SPEED_TOL AND 
                    CurrentPressure <= POLY_PRESS; (* TRUE if pressure <2 bar *)
    IF LogCount < 50 AND State <> Last_State THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Holding Reaction: Temp=', 
            CONCAT(TO_STRING(CurrentTemp), CONCAT(' °C, Press=', 
            CONCAT(TO_STRING(CurrentPressure), CONCAT(' bar, Speed=', 
            CONCAT(TO_STRING(CurrentAgitSpeed), ' RPM')))))));
    END_IF;
END_METHOD

(* Method: StartQuenching *)
(* Activates coolant, cools to target temperature *)
METHOD PRIVATE StartQuenching : BOOL
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := TRUE;
    HeaterControl := FALSE;
    CoolerControl := TRUE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := QUENCH_TEMP;      (* 30°C *)
    TargetAgitSpeed := 0.0;
    StartQuenching := ABS(CurrentTemp - QUENCH_TEMP) <= QUENCH_TEMP_TOL; (* TRUE if temp stable *)
    IF LogCount < 50 AND (CoolantValve <> Last_CoolantValve OR CoolerControl <> Last_CoolerControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Quenching Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), ' °C')));
    END_IF;
END_METHOD

(* Method: TransferToUnit *)
(* Activates conveyor for transfer *)
METHOD PRIVATE TransferToUnit : BOOL
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := TRUE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    TransferToUnit := CurrentWeight <= 0.0; (* TRUE if tank empty *)
    IF LogCount < 50 AND TransferConveyor <> Last_TransferConveyor THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Transfer Started');
    END_IF;
END_METHOD

(* Method: DryMaterial *)
(* Sets drying conditions, activates heater and agitator *)
METHOD PRIVATE DryMaterial : BOOL
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := TRUE;
    CoolerControl := FALSE;
    AgitatorControl := TRUE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := DRY_TEMP;         (* 90°C *)
    TargetAgitSpeed := DRY_SPEED;   (* 100 RPM *)
    DryMaterial := ABS(CurrentTemp - DRY_TEMP) <= DRY_TEMP_TOL AND 
                   ABS(CurrentAgitSpeed - DRY_SPEED) <= DRY_SPEED_TOL; (* TRUE if stable *)
    IF LogCount < 50 AND (TargetTemp <> Last_TargetTemp OR TargetAgitSpeed <> Last_TargetAgitSpeed OR 
                          HeaterControl <> Last_HeaterControl OR AgitatorControl <> Last_AgitatorControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Drying Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), CONCAT(' °C, Speed=', 
            CONCAT(TO_STRING(TargetAgitSpeed), ' RPM')))));
    END_IF;
END_METHOD

(* Method: Pelletize *)
(* Activates pelletizer at specified speed *)
METHOD PRIVATE Pelletize : BOOL
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := TRUE;
    PackagingControl := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := PELLET_SPEED; (* 200 RPM *)
    Pelletize := ABS(CurrentAgitSpeed - PELLET_SPEED) <= PELLET_SPEED_TOL; (* TRUE if speed stable *)
    IF LogCount < 50 AND PelletizerControl <> Last_PelletizerControl THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Pelletizing Started: TargetSpeed=', 
            CONCAT(TO_STRING(TargetAgitSpeed), ' RPM')));
    END_IF;
END_METHOD

(* Method: CheckQuality *)
(* Verifies product viscosity *)
METHOD PRIVATE CheckQuality : BOOL
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    CheckQuality := ABS(CurrentViscosity - VISCOSITY_TARGET) <= VISCOSITY_TOL; (* TRUE if 450–550 cP *)
    IF LogCount < 50 AND State <> Last_State THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Quality Check Started: TargetViscosity=', 
            CONCAT(TO_STRING(VISCOSITY_TARGET), ' cP')));
    END_IF;
END_METHOD

(* Method: PackageProduct *)
(* Activates packaging system *)
METHOD PRIVATE PackageProduct : BOOL
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := TRUE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    PackageProduct := CurrentWeight <= 0.0; (* TRUE if all packaged *)
    IF LogCount < 50 AND PackagingControl <> Last_PackagingControl THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Packaging Started');
    END_IF;
END_METHOD

(* Edge detection *)
StartBatch(CLK := NOT Last_StartBatch);
StopBatch(CLK := NOT Last_StopBatch);

(* Main Logic *)
(* State Machine Overview:
   - IDLE (0): Await StartBatch
   - DOSE_MONOMERS (1): Dose 300 kg monomers
   - DOSE_CATALYSTS (2): Dose 5 kg catalysts
   - DOSE_SOLVENTS (3): Dose 150 kg solvents
   - POLY_HEAT_AGITATE (4): Heat to 80°C, agitate at 120 RPM, 30m timeout
   - POLY_HOLD (5): Hold 80°C, 120 RPM until <2 bar, max 2h
   - QUENCH (6): Cool to 30°C, 20m timeout
   - DRY_TRANSFER (7): Transfer to dryer, 10m timeout
   - DRY_DRY (8): Dry at 90°C, 100 RPM, 1h
   - PELLETIZE (9): Pelletize at 200 RPM, 30m
   - QUALITY_CHECK (10): Check viscosity 500±50 cP, 5m
   - PACKAGE (11): Package in 50 kg bags, 10m
   - COMPLETED (12): Signal completion
   - ERROR (13): Safe state on fault
   - Methods: DoseIngredient, HeatToTemp, StartAgitation, etc.
*)

(* Step 1: Input validation *)
IF EmergencyStop THEN
    State := 13; (* ERROR *)
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    BatchComplete := FALSE;
    PhaseTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Emergency Stop: Switching to ERROR');
    END_IF;
    RETURN;
END_IF;

IF NOT IS_VALID_REAL(CurrentWeight) OR NOT IS_VALID_REAL(CurrentTemp) OR 
   NOT IS_VALID_REAL(CurrentPressure) OR NOT IS_VALID_REAL(CurrentAgitSpeed) OR 
   NOT IS_VALID_REAL(CurrentViscosity) OR
   CurrentWeight < MIN_WEIGHT OR CurrentWeight > MAX_WEIGHT OR
   CurrentTemp < MIN_TEMP OR CurrentTemp > MAX_TEMP OR
   CurrentPressure < MIN_PRESS OR CurrentPressure > MAX_PRESS OR
   CurrentAgitSpeed < MIN_SPEED OR CurrentAgitSpeed > MAX_SPEED OR
   CurrentViscosity < MIN_VISCOSITY OR CurrentViscosity > MAX_VISCOSITY THEN
    State := 13; (* ERROR *)
    MonomerValve := FALSE;
    CatalystValve := FALSE;
    SolventValve := FALSE;
    CoolantValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferConveyor := FALSE;
    PelletizerControl := FALSE;
    PackagingControl := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    BatchComplete := FALSE;
    PhaseTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:35:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Invalid Input: Weight=', 
            CONCAT(TO_STRING(CurrentWeight), CONCAT(' kg, Temp=', 
            CONCAT(TO_STRING(CurrentTemp), CONCAT(' °C, Press=', 
            CONCAT(TO_STRING(CurrentPressure), CONCAT(' bar, Speed=', 
            CONCAT(TO_STRING(CurrentAgitSpeed), CONCAT(' RPM, Viscosity=',
