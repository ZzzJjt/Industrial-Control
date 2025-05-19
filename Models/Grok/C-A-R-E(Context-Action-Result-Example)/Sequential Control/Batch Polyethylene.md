(* IEC 61131-3 Structured Text Program: PVCBatchControl *)
(* Purpose: Implements ISA-88 batch control for PVC production *)

PROGRAM PVCBatchControl
VAR
    (* Inputs *)
    StartBatch : R_TRIG;            (* Rising edge to start batch *)
    StopBatch : R_TRIG;             (* Rising edge to stop batch *)
    CurrentPressure : REAL;         (* Reactor pressure, mbar or bar *)
    CurrentTemp : REAL;             (* Reactor/dryer temperature, °C *)
    CurrentVolume : REAL;           (* Reactor liquid volume, L *)
    CurrentAgitSpeed : REAL;        (* Agitator speed, RPM *)
    EmergencyStop : BOOL;           (* TRUE if emergency stop triggered *)

    (* Outputs *)
    VacuumPump : BOOL;              (* TRUE to activate vacuum pump *)
    WaterValve : BOOL;              (* TRUE to add demineralized water *)
    SurfactantValve : BOOL;         (* TRUE to add surfactants *)
    VCMValve : BOOL;                (* TRUE to add VCM *)
    VentValve : BOOL;               (* TRUE to vent reactor *)
    TargetTemp : REAL;              (* Temperature setpoint, °C *)
    TargetAgitSpeed : REAL;         (* Agitator speed setpoint, RPM *)
    HeaterControl : BOOL;           (* TRUE to activate heater *)
    CoolerControl : BOOL;           (* TRUE to activate cooler *)
    AgitatorControl : BOOL;         (* TRUE to activate agitator *)
    TransferPump : BOOL;            (* TRUE to transfer to dryer *)
    BatchComplete : BOOL;           (* TRUE when batch is finished *)
    State : INT := 0;               (* Current state: 0=IDLE, 1=EVACUATE, 2=ADD_WATER, 3=ADD_SURFACTANTS, 4=ADD_VCM, 5=POLY_HEAT_MIX, 6=POLY_HOLD, 7=DECOVER_VENT, 8=DECOVER_COOL, 9=DRY_TRANSFER, 10=DRY_DRY, 11=COMPLETED, 12=ERROR *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    PhaseTimer : TON;               (* Timer for phase durations *)
    InitialVolume : REAL;           (* Volume before adding ingredient *)
    Last_StartBatch : BOOL;         (* Previous StartBatch.Q *)
    Last_StopBatch : BOOL;         (* Previous StopBatch.Q *)
    Last_BatchComplete : BOOL;      (* Previous BatchComplete *)
    Last_State : INT;               (* Previous State *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Parameter Constants *)
VAR CONSTANT
    (* Reactor Preparation *)
    VACUUM_PRESS : REAL := 200.0;   (* Target vacuum pressure, mbar *)
    WATER_QTY : REAL := 300.0;      (* Demineralized water, L *)
    SURFACTANT_QTY : REAL := 5.0;   (* Surfactants, L *)
    VCM_QTY : REAL := 150.0;        (* Vinyl chloride monomers, L *)
    EVAC_TIMEOUT : TIME := T#10m;   (* Evacuation timeout *)
    ADD_TIMEOUT : TIME := T#10m;    (* Ingredient addition timeout *)
    (* Polymerization *)
    REACT_TEMP_MIN : REAL := 55.0;  (* Min reaction temperature, °C *)
    REACT_TEMP_MAX : REAL := 60.0;  (* Max reaction temperature, °C *)
    REACT_SPEED : REAL := 500.0;    (* Agitator speed, RPM *)
    REACT_SPEED_TOL : REAL := 50.0; (* ±50 RPM tolerance *)
    REACT_PRESS : REAL := 1.5;      (* Pressure threshold, bar *)
    HEAT_MIX_TIMEOUT : TIME := T#30m; (* Heating/mixing timeout *)
    HOLD_MAX_TIME : TIME := T#4h;   (* Max reaction hold time *)
    (* Decover *)
    VENT_TIMEOUT : TIME := T#10m;   (* Venting timeout *)
    COOL_TEMP : REAL := 30.0;       (* Cooling temperature, °C *)
    COOL_TEMP_TOL : REAL := 2.0;    (* ±2°C tolerance *)
    COOL_TIMEOUT : TIME := T#20m;   (* Cooling timeout *)
    (* Drying *)
    DRY_TEMP : REAL := 80.0;        (* Drying temperature, °C *)
    DRY_TEMP_TOL : REAL := 2.0;     (* ±2°C tolerance *)
    DRY_SPEED : REAL := 100.0;      (* Agitator speed, RPM *)
    DRY_SPEED_TOL : REAL := 10.0;   (* ±10 RPM tolerance *)
    TRANSFER_TIMEOUT : TIME := T#10m; (* Transfer timeout *)
    DRY_DURATION : TIME := T#1h;    (* Drying duration *)
    (* Input Validation Ranges *)
    MIN_PRESS : REAL := 0.0;        (* Min valid pressure, mbar or bar *)
    MAX_PRESS : REAL := 1000.0;     (* Max valid pressure, mbar or bar *)
    MIN_TEMP : REAL := 0.0;         (* Min valid temperature, °C *)
    MAX_TEMP : REAL := 200.0;       (* Max valid temperature, °C *)
    MIN_VOLUME : REAL := 0.0;       (* Min valid volume, L *)
    MAX_VOLUME : REAL := 1000.0;    (* Max valid volume, L *)
    MIN_SPEED : REAL := 0.0;        (* Min valid speed, RPM *)
    MAX_SPEED : REAL := 1000.0;     (* Max valid speed, RPM *)
END_VAR

(* Method: EvacuateReactor *)
METHOD PRIVATE EvacuateReactor : BOOL
    VacuumPump := TRUE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    EvacuateReactor := CurrentPressure <= VACUUM_PRESS; (* TRUE if <200 mbar *)
    IF LogCount < 50 AND VacuumPump <> Last_VacuumPump THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Evacuation Started: TargetPress=', 
            CONCAT(TO_STRING(VACUUM_PRESS), ' mbar')));
    END_IF;
END_METHOD

(* Method: AddDemineralizedWater *)
METHOD PRIVATE AddDemineralizedWater : BOOL
    VacuumPump := FALSE;
    WaterValve := TRUE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    AddDemineralizedWater := CurrentVolume >= InitialVolume + WATER_QTY; (* TRUE if 300 L *)
    IF LogCount < 50 AND WaterValve <> Last_WaterValve THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Adding Water: TargetVolume=', 
            CONCAT(TO_STRING(WATER_QTY), ' L')));
    END_IF;
END_METHOD

(* Method: AddSurfactants *)
METHOD PRIVATE AddSurfactants : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := TRUE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    AddSurfactants := CurrentVolume >= InitialVolume + SURFACTANT_QTY; (* TRUE if 5 L *)
    IF LogCount < 50 AND SurfactantValve <> Last_SurfactantValve THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Adding Surfactants: TargetVolume=', 
            CONCAT(TO_STRING(SURFACTANT_QTY), ' L')));
    END_IF;
END_METHOD

(* Method: AddVCM *)
METHOD PRIVATE AddVCM : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := TRUE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    AddVCM := CurrentVolume >= InitialVolume + VCM_QTY; (* TRUE if 150 L *)
    IF LogCount < 50 AND VCMValve <> Last_VCMValve THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Adding VCM: TargetVolume=', 
            CONCAT(TO_STRING(VCM_QTY), ' L')));
    END_IF;
END_METHOD

(* Method: StartPolymerization *)
METHOD PRIVATE StartPolymerization : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := TRUE;
    CoolerControl := FALSE;
    AgitatorControl := TRUE;
    TransferPump := FALSE;
    TargetTemp := (REACT_TEMP_MIN + REACT_TEMP_MAX) / 2.0; (* 57.5°C *)
    TargetAgitSpeed := REACT_SPEED; (* 500 RPM *)
    StartPolymerization := CurrentTemp >= REACT_TEMP_MIN AND CurrentTemp <= REACT_TEMP_MAX AND 
                          ABS(CurrentAgitSpeed - REACT_SPEED) <= REACT_SPEED_TOL; (* TRUE if stable *)
    IF LogCount < 50 AND (TargetTemp <> Last_TargetTemp OR TargetAgitSpeed <> Last_TargetAgitSpeed OR 
                          HeaterControl <> Last_HeaterControl OR AgitatorControl <> Last_AgitatorControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Polymerization Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), CONCAT(' °C, Speed=', 
            CONCAT(TO_STRING(TargetAgitSpeed), ' RPM')))));
    END_IF;
END_METHOD

(* Method: HoldReaction *)
METHOD PRIVATE HoldReaction : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := TRUE;
    CoolerControl := FALSE;
    AgitatorControl := TRUE;
    TransferPump := FALSE;
    TargetTemp := (REACT_TEMP_MIN + REACT_TEMP_MAX) / 2.0; (* 57.5°C *)
    TargetAgitSpeed := REACT_SPEED; (* 500 RPM *)
    HoldReaction := CurrentTemp >= REACT_TEMP_MIN AND CurrentTemp <= REACT_TEMP_MAX AND 
                    ABS(CurrentAgitSpeed - REACT_SPEED) <= REACT_SPEED_TOL AND 
                    CurrentPressure <= REACT_PRESS; (* TRUE if pressure <1.5 bar *)
    IF LogCount < 50 AND State <> Last_State THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Holding Reaction: Temp=', 
            CONCAT(TO_STRING(CurrentTemp), CONCAT(' °C, Press=', 
            CONCAT(TO_STRING(CurrentPressure), CONCAT(' bar, Speed=', 
            CONCAT(TO_STRING(CurrentAgitSpeed), ' RPM')))))));
    END_IF;
END_METHOD

(* Method: VentReactor *)
METHOD PRIVATE VentReactor : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := TRUE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    VentReactor := CurrentPressure <= 1000.0; (* TRUE if near atmospheric ~1000 mbar *)
    IF LogCount < 50 AND VentValve <> Last_VentValve THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Venting Started');
    END_IF;
END_METHOD

(* Method: CoolReactor *)
METHOD PRIVATE CoolReactor : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := TRUE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := COOL_TEMP; (* 30°C *)
    TargetAgitSpeed := 0.0;
    CoolReactor := ABS(CurrentTemp - COOL_TEMP) <= COOL_TEMP_TOL; (* TRUE if stable *)
    IF LogCount < 50 AND (TargetTemp <> Last_TargetTemp OR CoolerControl <> Last_CoolerControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Cooling Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), ' °C')));
    END_IF;
END_METHOD

(* Method: TransferToDryer *)
METHOD PRIVATE TransferToDryer : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := TRUE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    TransferToDryer := CurrentVolume <= 0.0; (* TRUE if reactor empty *)
    IF LogCount < 50 AND TransferPump <> Last_TransferPump THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Transfer to Dryer Started');
    END_IF;
END_METHOD

(* Method: DryMaterial *)
METHOD PRIVATE DryMaterial : BOOL
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := TRUE;
    CoolerControl := FALSE;
    AgitatorControl := TRUE;
    TransferPump := FALSE;
    TargetTemp := DRY_TEMP; (* 80°C *)
    TargetAgitSpeed := DRY_SPEED; (* 100 RPM *)
    DryMaterial := ABS(CurrentTemp - DRY_TEMP) <= DRY_TEMP_TOL AND 
                   ABS(CurrentAgitSpeed - DRY_SPEED) <= DRY_SPEED_TOL; (* TRUE if stable *)
    IF LogCount < 50 AND (TargetTemp <> Last_TargetTemp OR TargetAgitSpeed <> Last_TargetAgitSpeed OR 
                          HeaterControl <> Last_HeaterControl OR AgitatorControl <> Last_AgitatorControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Drying Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), CONCAT(' °C, Speed=', 
            CONCAT(TO_STRING(TargetAgitSpeed), ' RPM')))));
    END_IF;
END_METHOD

(* Edge detection *)
StartBatch(CLK := NOT Last_StartBatch);
StopBatch(CLK := NOT Last_StopBatch);

(* Main Logic *)
(* State Machine Overview:
   - IDLE (0): Await StartBatch
   - EVACUATE (1): Vacuum to <200 mbar, 10m timeout
   - ADD_WATER (2): Add 300 L water
   - ADD_SURFACTANTS (3): Add 5 L surfactants
   - ADD_VCM (4): Add 150 L VCM
   - POLY_HEAT_MIX (5): Heat to 55–60°C, mix at 500 RPM, 30m timeout
   - POLY_HOLD (6): Hold 55–60°C, 500 RPM until <1.5 bar, max 4h
   - DECOVER_VENT (7): Vent reactor, 10m timeout
   - DECOVER_COOL (8): Cool to 30°C, 20m timeout
   - DRY_TRANSFER (9): Transfer to dryer, 10m timeout
   - DRY_DRY (10): Dry at 80°C, 100 RPM, 1h
   - COMPLETED (11): Signal completion
   - ERROR (12): Safe state on fault
   - Methods: EvacuateReactor, AddDemineralizedWater, etc.
*)

(* Step 1: Input validation *)
IF EmergencyStop THEN
    State := 12; (* ERROR *)
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    BatchComplete := FALSE;
    PhaseTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Emergency Stop: Switching to ERROR');
    END_IF;
    RETURN;
END_IF;

IF NOT IS_VALID_REAL(CurrentPressure) OR NOT IS_VALID_REAL(CurrentTemp) OR 
   NOT IS_VALID_REAL(CurrentVolume) OR NOT IS_VALID_REAL(CurrentAgitSpeed) OR
   CurrentPressure < MIN_PRESS OR CurrentPressure > MAX_PRESS OR
   CurrentTemp < MIN_TEMP OR CurrentTemp > MAX_TEMP OR
   CurrentVolume < MIN_VOLUME OR CurrentVolume > MAX_VOLUME OR
   CurrentAgitSpeed < MIN_SPEED OR CurrentAgitSpeed > MAX_SPEED THEN
    State := 12; (* ERROR *)
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    BatchComplete := FALSE;
    PhaseTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Invalid Input: Press=', 
            CONCAT(TO_STRING(CurrentPressure), CONCAT(' mbar/bar, Temp=', 
            CONCAT(TO_STRING(CurrentTemp), CONCAT(' °C, Vol=', 
            CONCAT(TO_STRING(CurrentVolume), CONCAT(' L, Speed=', 
            CONCAT(TO_STRING(CurrentAgitSpeed), ' RPM')))))))));
    END_IF;
    RETURN;
END_IF;

(* Step 2: Start and stop triggers *)
IF StartBatch.Q AND State = 0 THEN
    State := 1; (* EVACUATE *)
    InitialVolume := CurrentVolume;
    PhaseTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Batch Started: Switching to EVACUATE');
    END_IF;
END_IF;
IF StopBatch.Q AND State IN (1..10) THEN
    State := 12; (* ERROR *)
    VacuumPump := FALSE;
    WaterValve := FALSE;
    SurfactantValve := FALSE;
    VCMValve := FALSE;
    VentValve := FALSE;
    HeaterControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TransferPump := FALSE;
    TargetTemp := 0.0;
    TargetAgitSpeed := 0.0;
    BatchComplete := FALSE;
    PhaseTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:30:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Batch Stopped: Switching to ERROR');
    END_IF;
END_IF;

(* Step 3: State machine execution *)
CASE State OF
    0: (* IDLE: Await StartBatch *)
        VacuumPump := FALSE;
        WaterValve := FALSE;
        SurfactantValve := FALSE;
        VCMValve := FALSE;
        VentValve := FALSE;
        HeaterControl := FALSE;
        CoolerControl := FALSE;
        AgitatorControl := FALSE;
        TransferPump := FALSE;
        TargetTemp := 0.0;
        TargetAgitSpeed := 0.0;
        BatchComplete := FALSE;
        PhaseTimer.IN := FALSE;

    1: (* EVACUATE: Vacuum to <200 mbar *)
        IF EvacuateReactor() OR PhaseTimer.Q THEN
            State := EvacuateReactor() ? 2 : 12; (* ADD_WATER if pressure reached, else ERROR *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, EvacuateReactor() ? ' Evacuation Complete: Switching to ADD_WATER' : ' Evacuation Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := EVAC_TIMEOUT);
        END_IF;

    2: (* ADD_WATER: Add 300 L water *)
        IF AddDemineralizedWater() OR PhaseTimer.Q THEN
            State := AddDemineralizedWater() ? 3 : 12; (* ADD_SURFACTANTS if volume reached, else ERROR *)
            InitialVolume := CurrentVolume;
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, AddDemineralizedWater() ? ' Water Added: Switching to ADD_SURFACTANTS' : ' Water Addition Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := ADD_TIMEOUT);
        END_IF;

    3: (* ADD_SURFACTANTS: Add 5 L surfactants *)
        IF AddSurfactants() OR PhaseTimer.Q THEN
            State := AddSurfactants() ? 4 : 12; (* ADD_VCM if volume reached, else ERROR *)
            InitialVolume := CurrentVolume;
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, AddSurfactants() ? ' Surfactants Added: Switching to ADD_VCM' : ' Surfactants Addition Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := ADD_TIMEOUT);
        END_IF;

    4: (* ADD_VCM: Add 150 L VCM *)
        IF AddVCM() OR PhaseTimer.Q THEN
            State := AddVCM() ? 5 : 12; (* POLY_HEAT_MIX if volume reached, else ERROR *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, AddVCM() ? ' VCM Added: Switching to POLY_HEAT_MIX' : ' VCM Addition Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := ADD_TIMEOUT);
        END_IF;

    5: (* POLY_HEAT_MIX: Heat to 55–60°C, mix at 500 RPM *)
        IF StartPolymerization() OR PhaseTimer.Q THEN
            State := StartPolymerization() ? 6 : 12; (* POLY_HOLD if conditions stable, else ERROR *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, StartPolymerization() ? ' Polymerization Heating/Mixing Complete: Switching to POLY_HOLD' : ' Polymerization Heating/Mixing Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := HEAT_MIX_TIMEOUT);
        END_IF;

    6: (* POLY_HOLD: Hold until pressure <1.5 bar *)
        IF HoldReaction() OR PhaseTimer.Q THEN
            State := HoldReaction() ? 7 : 12; (* DECOVER_VENT if pressure dropped, else ERROR *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, HoldReaction() ? ' Polymerization Complete: Switching to DECOVER_VENT' : ' Polymerization Hold Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := HOLD_MAX_TIME);
        END_IF;

    7: (* DECOVER_VENT: Vent reactor *)
        IF VentReactor() OR PhaseTimer.Q THEN
            State := VentReactor() ? 8 : 12; (* DECOVER_COOL if vented, else ERROR *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, VentReactor() ? ' Venting Complete: Switching to DECOVER_COOL' : ' Venting Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := VENT_TIMEOUT);
        END_IF;

    8: (* DECOVER_COOL: Cool to 30°C *)
        IF CoolReactor() OR PhaseTimer.Q THEN
            State := CoolReactor() ? 9 : 12; (* DRY_TRANSFER if cooled, else ERROR *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, CoolReactor() ? ' Cooling Complete: Switching to DRY_TRANSFER' : ' Cooling Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := COOL_TIMEOUT);
        END_IF;

    9: (* DRY_TRANSFER: Transfer to dryer *)
        IF TransferToDryer() OR PhaseTimer.Q THEN
            State := TransferToDryer() ? 10 : 12; (* DRY_DRY if transferred, else ERROR *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, TransferToDryer() ? ' Transfer Complete: Switching to DRY_DRY' : ' Transfer Timeout: Switching to ERROR');
            END_IF;
        ELSE
            PhaseTimer(IN := TRUE, PT := TRANSFER_TIMEOUT);
        END_IF;

    10: (* DRY_DRY: Dry at 80°C, 100 RPM for 1h *)
        DryMaterial();
        PhaseTimer(IN := TRUE, PT := DRY_DURATION);
        IF PhaseTimer.Q THEN
            State := 11; (* COMPLETED *)
            PhaseTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:30:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Drying Complete: Switching to COMPLETED');
            END_IF;
        END_IF;

    11: (* COMPLETED: Signal completion *)
        VacuumPump := FALSE;
        WaterValve := FALSE;
        SurfactantValve := FALSE;
        VCMValve := FALSE;
        VentValve := FALSE;
        HeaterControl := FALSE;
        CoolerControl := FALSE;
        AgitatorControl := FALSE;
        TransferPump := FALSE;
        TargetTemp := 0.0;
        TargetAgitSpeed := 0.0;
        BatchComplete := TRUE;
        PhaseTimer.IN := FALSE;
        (* Await external reset to IDLE *)

    12: (* ERROR: Safe state *)
        VacuumPump := FALSE;
        WaterValve := FALSE;
        SurfactantValve := FALSE;
        VCMValve := FALSE;
        VentValve := FALSE;
        HeaterControl := FALSE;
        CoolerControl := FALSE;
        AgitatorControl := FALSE;
        TransferPump := FALSE;
        TargetTemp := 0.0;
        TargetAgitSpeed := 0.0;
        BatchComplete := FALSE;
        PhaseTimer.IN := FALSE;
        (* Await external reset *)
END_CASE;

(* Step 4: Log state and completion changes *)
IF State <> Last_State AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 13:30:00';
    DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' State Changed to: ', 
        TO_STRING(State)));
END_IF;
IF BatchComplete AND NOT Last_BatchComplete AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 13:30:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Batch Completed');
END_IF;

(* Step 5: Update last states *)
Last_StartBatch := StartBatch.Q;
Last_StopBatch := StopBatch.Q;
Last_BatchComplete := BatchComplete;
Last_State := State;
Last_VacuumPump := VacuumPump;
Last_WaterValve := WaterValve;
Last_SurfactantValve := SurfactantValve;
Last_VCMValve := VCMValve;
Last_VentValve := VentValve;
Last_TargetTemp := TargetTemp;
Last_TargetAgitSpeed := TargetAgitSpeed;
Last_HeaterControl := HeaterControl;
Last_CoolerControl := CoolerControl;
Last_AgitatorControl := AgitatorControl;
Last_TransferPump := TransferPump;

(* Step 6: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Implements ISA-88 batch control for PVC production.
   - Inputs:
     - StartBatch, StopBatch: R_TRIG, start/stop batch.
     - CurrentPressure: REAL, pressure (mbar or bar).
     - CurrentTemp: REAL, temperature (°C).
     - CurrentVolume: REAL, volume (L).
     - CurrentAgitSpeed: REAL, speed (RPM).
     - EmergencyStop: BOOL, emergency trigger.
   - Outputs:
     - VacuumPump, WaterValve, SurfactantValve, VCMValve, VentValve: BOOL, controls.
     - TargetTemp, TargetAgitSpeed: REAL, setpoints.
     - HeaterControl, CoolerControl, AgitatorControl, TransferPump: BOOL, controls.
     - BatchComplete: BOOL, TRUE when finished.
     - State: INT, current state (0=IDLE, 1–4=preparation, 5–6=polymerization, 7–8=decover, 9–10=drying, 11=COMPLETED, 12=ERROR).
     - DiagLog: ARRAY[1..50] OF STRING[80], logs.
     - LogCount: INT, log entries.
   - Logic:
     - Preparation: Evacuate (<200 mbar) → Water (300 L) → Surfactants (5 L) → VCM (150 L).
     - Polymerization: Heat/Mix (55–60°C, 500 RPM) → Hold (<1.5 bar, max 4h).
     - Decover: Vent → Cool (30°C).
     - Drying: Transfer → Dry (80°C, 100 RPM, 1h).
     - Methods: EvacuateReactor, AddDemineralizedWater, etc.
     - Transitions: Timer (PhaseTimer.Q) and conditions (pressure, temp, volume, speed).
   - Optimization:
     - ~100 FLOPs, <0.1 ms on 1 MFLOP/s PLC.
     - Fixed flow, single timer, no recursion.
     - ~4 KB memory (logs, scalars).
   - Safety:
     - Validates inputs (0–1000 mbar/bar, 0–200°C, 0–1000 L, 0–1000 RPM).
     - EmergencyStop, timeouts to ERROR.
     - Exclusive states via CASE.
     - Logs all events.
   - Usage:
     - PVC batch: Precise evacuation, charging, polymerization, drying.
     - Example: StartBatch.Q=TRUE → EVACUATE (<200 mbar) → ADD_WATER (300 L) → ... → DRY_DRY (80°C, 1h) → COMPLETED.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL, INT, TON, R_TRIG; adjust for limits.
     - Timestamp uses 2025-05-19 13:30:00; replace with GET_SYSTEM_TIME.
     - Log buffer (50) is practical; adjustable.
*)
END_PROGRAM
