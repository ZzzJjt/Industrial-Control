(* ISA-88 Compliant Structured Text Program for Urea Fertilizer Production *)
(* Unit Procedure: Reaction Stage *)
(* Conforms to IEC 61131-3 Standard *)

PROGRAM UreaReactionControl
VAR
    (* ISA-88 Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: Heating, 2: PressureRegulation, 3: Cooling, 4: Complete, 5: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters *)
    HeatingTempSetpoint: REAL := 180.0; (* Target heating temperature in °C *)
    TempTolerance: REAL := 2.0; (* Acceptable temperature deviation *)
    PressureSetpoint: REAL := 140.0; (* Target reaction pressure in bar *)
    PressureTolerance: REAL := 5.0; (* Acceptable pressure deviation *)
    ReactionHoldTime: TIME := T#30m; (* Reaction hold duration *)
    CoolingTempSetpoint: REAL := 80.0; (* Target cooling temperature in °C *)
    MaxPhaseTime: TIME := T#60m; (* Maximum phase duration *)
    
    (* Process Inputs *)
    CurrentTemp: REAL; (* Current reactor temperature *)
    CurrentPressure: REAL; (* Current reactor pressure *)
    
    (* Process Outputs *)
    HeaterOn: BOOL; (* Reactor heater control *)
    PressureValve: BOOL; (* Pressure control valve *)
    CoolingValve: BOOL; (* Cooling system valve *)
    ReactionComplete: BOOL; (* Reaction phase completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Timer for overall phase duration *)
    HoldTimer: TON; (* Timer for pressure regulation phase *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
END_VAR

(* Modular Function Blocks *)
FUNCTION_BLOCK StartHeating
    VAR_INPUT
        TempSetpoint: REAL; (* Target temperature *)
        CurrentTemp: REAL; (* Measured temperature *)
        Tolerance: REAL; (* Acceptable deviation *)
    END_VAR
    VAR_OUTPUT
        HeaterCmd: BOOL; (* Heater control signal *)
        HeatingComplete: BOOL; (* Heating completion flag *)
    END_VAR
    (* Logic to control heating to target temperature *)
    HeaterCmd := (CurrentTemp < (TempSetpoint - Tolerance));
    HeatingComplete := (ABS(CurrentTemp - TempSetpoint) <= Tolerance);
END_FUNCTION_BLOCK

FUNCTION_BLOCK RegulatePressure
    VAR_INPUT
        PressureSetpoint: REAL; (* Target pressure *)
        CurrentPressure: REAL; (* Measured pressure *)
        PressureTolerance: REAL; (* Acceptable deviation *)
        CurrentTemp: REAL; (* Measured temperature *)
        TempSetpoint: REAL; (* Target temperature *)
        TempTolerance: REAL; (* Acceptable temperature deviation *)
        Timer: TON; (* Reaction timer *)
        Duration: TIME; (* Reaction duration *)
    END_VAR
    VAR_OUTPUT
        ValveCmd: BOOL; (* Pressure valve control *)
        HeaterCmd: BOOL; (* Heater control to maintain temperature *)
        ReactionComplete: BOOL; (* Reaction completion flag *)
    END_VAR
    (* Logic to maintain pressure and temperature during reaction *)
    ValveCmd := (CurrentPressure < (PressureSetpoint - PressureTolerance));
    HeaterCmd := (CurrentTemp < (TempSetpoint - TempTolerance));
    Timer(IN := (ABS(CurrentPressure - PressureSetpoint) <= PressureTolerance) AND
                (ABS(CurrentTemp - TempSetpoint) <= TempTolerance), 
          PT := Duration);
    ReactionComplete := Timer.Q;
END_FUNCTION_BLOCK

FUNCTION_BLOCK StartCooling
    VAR_INPUT
        TempSetpoint: REAL; (* Target cooling temperature *)
        CurrentTemp: REAL; (* Measured temperature *)
        Tolerance: REAL; (* Acceptable deviation *)
    END_VAR
    VAR_OUTPUT
        CoolingCmd: BOOL; (* Cooling valve control *)
        CoolingComplete: BOOL; (* Cooling completion flag *)
    END_VAR
    (* Logic to cool reactor to target temperature *)
    CoolingCmd := (CurrentTemp > (TempSetpoint + Tolerance));
    CoolingComplete := (ABS(CurrentTemp - TempSetpoint) <= Tolerance);
END_FUNCTION_BLOCK

(* Main Phase Control Logic *)
VAR
    HeatingFB: StartHeating;
    PressureFB: RegulatePressure;
    CoolingFB: StartCooling;
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        (* Wait for start command, ensure no faults or emergency stop *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1; (* Initiate heating *)
            PhaseTimer(IN := TRUE, PT := MaxPhaseTime);
        END_IF;
    
    1: (* Heating Phase *)
        (* Heat reactor to 180°C *)
        HeatingFB(TempSetpoint := HeatingTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        HeaterOn := HeatingFB.HeaterCmd;
        
        (* Transition to pressure regulation *)
        IF HeatingFB.HeatingComplete AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransitionDelay(IN := FALSE);
        ELSIF HeatingFB.HeatingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Pressure Regulation Phase *)
        (* Maintain 140 bar and 180°C for 30 minutes *)
        PressureFB(PressureSetpoint := PressureSetpoint, 
                   CurrentPressure := CurrentPressure, 
                   PressureTolerance := PressureTolerance, 
                   CurrentTemp := CurrentTemp, 
                   TempSetpoint := HeatingTempSetpoint, 
                   TempTolerance := TempTolerance, 
                   Timer := HoldTimer, 
                   Duration := ReactionHoldTime);
        PressureValve := PressureFB.ValveCmd;
        HeaterOn := PressureFB.HeaterCmd;
        
        (* Transition to cooling *)
        IF PressureFB.ReactionComplete AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransitionDelay(IN := FALSE);
        ELSIF PressureFB.ReactionComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Cooling Phase *)
        (* Cool reactor to 80°C *)
        CoolingFB(TempSetpoint := CoolingTempSetpoint, 
                  CurrentTemp := CurrentTemp, 
                  Tolerance := TempTolerance);
        CoolingValve := CoolingFB.CoolingCmd;
        
        (* Transition to complete *)
        IF CoolingFB.CoolingComplete AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransitionDelay(IN := FALSE);
            ReactionComplete := TRUE;
        ELSIF CoolingFB.CoolingComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Complete State *)
        (* Reset outputs and signal completion *)
        HeaterOn := FALSE;
        PressureValve := FALSE;
        CoolingValve := FALSE;
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
        PressureValve := FALSE;
        CoolingValve := FALSE;
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

(* Temperature and Pressure Monitoring *)
IF (CurrentTemp > (HeatingTempSetpoint + 10.0) OR 
    CurrentTemp < (HeatingTempSetpoint - 10.0)) AND PhaseState IN (1..2) THEN
    FaultState := TRUE;
    PhaseState := 5;
END_IF;

IF (CurrentPressure > (PressureSetpoint + 10.0) OR 
    CurrentPressure < (PressureSetpoint - 10.0)) AND PhaseState = 2 THEN
    FaultState := TRUE;
    PhaseState := 5;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 5;
END_IF;

(* Synchronization Interlock *)
(* Ensure temperature and pressure are within tolerance before advancing *)
IF PhaseState = 2 AND 
   (ABS(CurrentTemp - HeatingTempSetpoint) > TempTolerance OR 
    ABS(CurrentPressure - PressureSetpoint) > PressureTolerance) THEN
    HoldTimer(IN := FALSE); (* Pause reaction timer if out of sync *)
ELSE
    HoldTimer(IN := TRUE);
END_IF;

END_PROGRAM
