(* ISA-88 Compliant Structured Text Program for Steel Furnace Shutdown *)
(* Unit Procedure: Furnace Shutdown *)
(* Conforms to IEC 61131-3 Standard *)

PROGRAM SteelFurnaceShutdownControl
VAR
    (* ISA-88 Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: Initiate, 2: TempReduction, 3: InitialFuelReduction, 4: MainFuelRampDown, 5: OxygenRegulation, 6: FinalCooling, 7: Complete, 8: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters *)
    InitialTempSetpoint: REAL := 800.0; (* Initial temperature reduction setpoint in °C *)
    FinalTempSetpoint: REAL := 400.0; (* Final cooling setpoint in °C *)
    TempTolerance: REAL := 10.0; (* Acceptable temperature deviation *)
    MaxCoolingRate: REAL := 50.0; (* Max cooling rate in °C/hour *)
    FuelFlowInitial: REAL := 10000.0; (* Initial fuel gas flow in Nm³/h *)
    FuelFlowMainStart: REAL := 5000.0; (* Fuel flow at start of main ramp-down *)
    FuelRampDuration: TIME := T#12h; (* Main fuel ramp-down duration *)
    FuelToAirRatio: REAL := 2.5; (* Air flow = Fuel flow * 2.5 *)
    CombustionEfficiencyMin: REAL := 90.0; (* Minimum combustion efficiency in % *)
    MaxPhaseTime: TIME := T#24h; (* Maximum shutdown duration *)
    MaxChamberPressure: REAL := 1.5; (* Max combustion chamber pressure in bar *)
    
    (* Process Inputs *)
    CurrentTemp: REAL; (* Current furnace temperature in °C *)
    CurrentFuelFlow: REAL; (* Current fuel gas flow in Nm³/h *)
    CurrentAirFlow: REAL; (* Current air flow in Nm³/h *)
    CurrentChamberPressure: REAL; (* Current combustion chamber pressure in bar *)
    CombustionEfficiency: REAL; (* Measured combustion efficiency in % *)
    MaterialFeedStopped: BOOL; (* Material feed confirmation *)
    
    (* Process Outputs *)
    HeaterOn: BOOL; (* Furnace heater control *)
    FuelValveSetpoint: REAL; (* Fuel valve setpoint in Nm³/h *)
    AirValveSetpoint: REAL; (* Air valve setpoint in Nm³/h *)
    CoolingWaterValve: BOOL; (* Cooling water valve control *)
    NitrogenPurgeValve: BOOL; (* Nitrogen purge valve *)
    ShutdownComplete: BOOL; (* Shutdown completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Timer for overall shutdown duration *)
    FuelRampTimer: TON; (* Timer for fuel ramp-down *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
    FuelRampInterval: TIME := T#1m; (* Fuel flow update interval *)
END_VAR

(* Function to Ramp Down Fuel Gas *)
FUNCTION RampDownFuelGas : REAL
    VAR_INPUT
        InitialFlow: REAL; (* Starting fuel flow in Nm³/h *)
        CurrentFlow: REAL; (* Current measured flow *)
        RampDuration: TIME; (* Total ramp-down duration *)
        Timer: TON; (* Ramp timer *)
        Interval: TIME; (* Update interval *)
    END_VAR
    VAR_OUTPUT
        FlowSetpoint: REAL; (* Calculated flow setpoint *)
        RampComplete: BOOL; (* Ramp completion flag *)
        Fault: BOOL; (* Fault flag *)
    END_VAR
    VAR
        ElapsedTime: TIME; (* Time elapsed since ramp start *)
        FlowReduction: REAL; (* Total flow to reduce *)
        FlowPerInterval: REAL; (* Flow reduction per interval *)
    END_VAR
    (* Logic to linearly ramp down fuel gas over 12 hours *)
    Timer(IN := TRUE, PT := RampDuration);
    ElapsedTime := Timer.ET;
    FlowReduction := InitialFlow; (* Total reduction from InitialFlow to 0 *)
    FlowPerInterval := FlowReduction / (TIME_TO_REAL(RampDuration) / TIME_TO_REAL(Interval));
    
    (* Calculate setpoint for current interval *)
    FlowSetpoint := InitialFlow - (FlowPerInterval * (TIME_TO_REAL(ElapsedTime) / TIME_TO_REAL(Interval)));
    FlowSetpoint := MAX(0.0, FlowSetpoint); (* Ensure non-negative *)
    
    (* Check for completion *)
    RampComplete := Timer.Q OR (FlowSetpoint <= 0.0);
    
    (* Fault if measured flow deviates >10% from setpoint *)
    Fault := (ABS(CurrentFlow - FlowSetpoint) > (FlowSetpoint * 0.1)) AND NOT RampComplete;
END_FUNCTION

(* Function to Adjust Oxygen Supply *)
FUNCTION AdjustOxygenSupply : REAL
    VAR_INPUT
        FuelFlow: REAL; (* Current fuel flow in Nm³/h *)
        CurrentTemp: REAL; (* Current furnace temperature *)
        TempThreshold: REAL; (* Temperature threshold for ratio adjustment *)
        NominalRatio: REAL; (* Nominal fuel-to-air ratio, e.g., 2.5 *)
    END_VAR
    VAR_OUTPUT
        AirFlowSetpoint: REAL; (* Calculated air flow setpoint *)
        Fault: BOOL; (* Fault flag *)
    END_VAR
    VAR
        AdjustedRatio: REAL; (* Adjusted ratio based on temperature *)
    END_VAR
    (* Logic to maintain 1:2.5 fuel-to-air ratio, adjust for low temp *)
    IF CurrentTemp < TempThreshold THEN
        AdjustedRatio := 2.0; (* Reduce air at low temp to prevent over-oxidation *)
    ELSE
        AdjustedRatio := NominalRatio;
    END_IF;
    
    AirFlowSetpoint := FuelFlow * AdjustedRatio;
    
    (* Fault if air flow cannot maintain combustion efficiency *)
    Fault := FALSE; (* Updated in main loop with CombustionEfficiency *)
END_FUNCTION

(* Main Phase Control Logic *)
VAR
    FuelRampResult: REAL; (* Fuel ramp-down setpoint *)
    FuelRampComplete: BOOL; (* Fuel ramp completion *)
    FuelRampFault: BOOL; (* Fuel ramp fault *)
    AirFlowResult: REAL; (* Air flow setpoint *)
    AirFlowFault: BOOL; (* Air flow fault *)
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        (* Wait for shutdown command *)
        IF ShutdownCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1;
            PhaseTimer(IN := TRUE, PT := MaxPhaseTime);
        END_IF;
    
    1: (* Initiate Shutdown *)
        (* Stop material feed *)
        MaterialFeedStopped := TRUE; (* Assume feed stop confirmed *)
        
        (* Transition to temperature reduction *)
        IF MaterialFeedStopped AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransitionDelay(IN := FALSE);
        ELSIF MaterialFeedStopped THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Temperature Reduction *)
        (* Reduce temperature to <800°C *)
        HeaterOn := (CurrentTemp > InitialTempSetpoint);
        CoolingWaterValve := (CurrentTemp > InitialTempSetpoint);
        
        (* Check cooling rate *)
        IF (CurrentTemp - InitialTempSetpoint) > MaxCoolingRate THEN
            FaultState := TRUE;
            PhaseState := 8;
        END_IF;
        
        (* Transition to initial fuel reduction *)
        IF CurrentTemp < InitialTempSetpoint AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransitionDelay(IN := FALSE);
        ELSIF CurrentTemp < InitialTempSetpoint THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Initial Fuel Reduction *)
        (* Reduce fuel flow to 50% *)
        FuelValveSetpoint := FuelFlowMainStart;
        
        (* Transition to main fuel ramp-down *)
        IF ABS(CurrentFuelFlow - FuelFlowMainStart) < (FuelFlowMainStart * 0.05) AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransitionDelay(IN := FALSE);
        ELSIF ABS(CurrentFuelFlow - FuelFlowMainStart) < (FuelFlowMainStart * 0.05) THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Main Fuel Ramp-Down *)
        (* Ramp down fuel gas from 50% to 0% over 12 hours *)
        FuelRampResult := RampDownFuelGas(
            InitialFlow := FuelFlowMainStart,
            CurrentFlow := CurrentFuelFlow,
            RampDuration := FuelRampDuration,
            Timer := FuelRampTimer,
            Interval := FuelRampInterval
        );
        FuelValveSetpoint := FuelRampResult.FlowSetpoint;
        FuelRampComplete := FuelRampResult.RampComplete;
        FuelRampFault := FuelRampResult.Fault;
        
        (* Transition to oxygen regulation *)
        IF FuelRampComplete AND TransitionDelay.Q THEN
            PhaseState := 5;
            TransitionDelay(IN := FALSE);
        ELSIF FuelRampComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
        
        (* Fault check *)
        IF FuelRampFault THEN
            FaultState := TRUE;
            PhaseState := 8;
        END_IF;
    
    5: (* Oxygen Regulation *)
        (* Adjust air flow to maintain 1:2.5 ratio *)
        AirFlowResult := AdjustOxygenSupply(
            FuelFlow := CurrentFuelFlow,
            CurrentTemp := CurrentTemp,
            TempThreshold := 600.0,
            NominalRatio := FuelToAirRatio
        );
        AirValveSetpoint := AirFlowResult.AirFlowSetpoint;
        AirFlowFault := AirFlowResult.Fault OR (CombustionEfficiency < CombustionEfficiencyMin);
        
        (* Interlock: Pause if chamber pressure too high *)
        IF CurrentChamberPressure > MaxChamberPressure THEN
            AirValveSetpoint := 0.0;
        END_IF;
        
        (* Transition to final cooling *)
        IF FuelRampComplete AND CurrentFuelFlow <= 0.0 AND TransitionDelay.Q THEN
            PhaseState := 6;
            TransitionDelay(IN := FALSE);
        ELSIF FuelRampComplete AND CurrentFuelFlow <= 0.0 THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
        
        (* Fault check *)
        IF AirFlowFault THEN
            FaultState := TRUE;
            PhaseState := 8;
        END_IF;
    
    6: (* Final Cooling and System Isolation *)
        (* Cool to <400°C and isolate system *)
        CoolingWaterValve := (CurrentTemp > FinalTempSetpoint);
        
        (* Check cooling rate *)
        IF (CurrentTemp - FinalTempSetpoint) > MaxCoolingRate THEN
            FaultState := TRUE;
            PhaseState := 8;
        END_IF;
        
        (* Isolate system *)
        IF CurrentTemp < FinalTempSetpoint AND CurrentFuelFlow = 0.0 AND CurrentAirFlow = 0.0 THEN
            FuelValveSetpoint := 0.0;
            AirValveSetpoint := 0.0;
            NitrogenPurgeValve := FALSE;
        END_IF;
        
        (* Transition to complete *)
        IF CurrentTemp < FinalTempSetpoint AND CurrentChamberPressure < 0.1 AND TransitionDelay.Q THEN
            PhaseState := 7;
            TransitionDelay(IN := FALSE);
            ShutdownComplete := TRUE;
        ELSIF CurrentTemp < FinalTempSetpoint AND CurrentChamberPressure < 0.1 THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    7: (* Complete State *)
        (* Reset outputs *)
        HeaterOn := FALSE;
        FuelValveSetpoint := 0.0;
        AirValveSetpoint := 0.0;
        CoolingWaterValve := FALSE;
        NitrogenPurgeValve := FALSE;
        PhaseTimer(IN := FALSE);
        FuelRampTimer(IN := FALSE);
        
        (* Wait for reset *)
        IF ResetCommand THEN
            PhaseState := 0;
            ShutdownComplete := FALSE;
        END_IF;
    
    8: (* Fault State *)
        (* Disable all outputs *)
        HeaterOn := FALSE;
        FuelValveSetpoint := 0.0;
        AirValveSetpoint := 0.0;
        CoolingWaterValve := FALSE;
        NitrogenPurgeValve := FALSE;
        PhaseTimer(IN := FALSE);
        FuelRampTimer(IN := FALSE);
        
        (* Require manual reset *)
        IF ResetCommand AND NOT EmergencyStop THEN
            PhaseState := 0;
            FaultState := FALSE;
        END_IF;
END_CASE;

(* Safety and Fault Monitoring *)
IF EmergencyStop THEN
    PhaseState := 8;
    FaultState := TRUE;
END_IF;

(* Temperature and Pressure Monitoring *)
IF CurrentTemp > (InitialTempSetpoint + 50.0) AND PhaseState IN (2..6) THEN
    FaultState := TRUE;
    PhaseState := 8;
END_IF;

IF CurrentChamberPressure > MaxChamberPressure + 0.5 AND PhaseState IN (3..5) THEN
    FaultState := TRUE;
    PhaseState := 8;
END_IF;

(* Combustion Efficiency Monitoring *)
IF CombustionEfficiency < CombustionEfficiencyMin AND PhaseState IN (4..5) THEN
    FaultState := TRUE;
    PhaseState := 8;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 8;
END_IF;

END_PROGRAM
