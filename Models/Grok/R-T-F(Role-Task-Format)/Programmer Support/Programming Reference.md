(*
  PolyethyleneBatchControl
  Purpose: Controls a polyethylene batch process with states for raw material preparation,
           reaction, cooling, and discharge.
  Author: [Your Name]
  Date: May 20, 2025
  Notes: 
  - Uses a state machine for cyclic execution, eliminating LOOP constructs.
  - Modularizes temperature/pressure control in helper methods.
  - Standardizes timer logic with TON for each state.
  - Ensures safety with input validation and fault handling.
*)

PROGRAM PolyethyleneBatchControl
VAR
    (* Inputs *)
    StartBatch : BOOL; (* TRUE to initiate batch process *)
    EmergencyStop : BOOL; (* TRUE if emergency stop is pressed *)
    EthyleneFlow_PV : REAL; (* Measured ethylene flow rate (L/min) *)
    CatalystFlow_PV : REAL; (* Measured catalyst flow rate (L/min) *)
    Pressure_PV : REAL; (* Measured reactor pressure (bar) *)
    Temperature_PV : REAL; (* Measured reactor temperature (°C) *)
    EthyleneValveStatus : BOOL; (* TRUE if ethylene valve is open *)
    CatalystValveStatus : BOOL; (* TRUE if catalyst valve is open *)
    DischargeValveStatus : BOOL; (* TRUE if discharge valve is open *)
    
    (* Outputs *)
    EthyleneValveCmd : BOOL; (* Command to open/close ethylene valve *)
    CatalystValveCmd : BOOL; (* Command to open/close catalyst valve *)
    DischargeValveCmd : BOOL; (* Command to open/close discharge valve *)
    BatchComplete : BOOL; (* TRUE when batch process is complete *)
    FaultAlarm : BOOL; (* TRUE if a fault is detected *)
    
    (* Internal variables *)
    State : INT := 0; (* State machine: 0=Idle, 1=RawMaterialPrep, 2=Reaction, 3=Cooling, 4=Discharge *)
    PhaseTimer : TON; (* Timer for state durations *)
    StartTrigger : R_TRIG; (* Rising edge detection for StartBatch *)
    
    (* Configurable parameters *)
    RawMatPrepDuration : TIME := T#30s; (* Duration for raw material preparation *)
    ReactionDuration : TIME := T#300s; (* Duration for reaction phase *)
    CoolingDuration : TIME := T#120s; (* Duration for cooling phase *)
    DischargeDuration : TIME := T#60s; (* Duration for discharge phase *)
    Pressure_SP : REAL := 50.0; (* Reaction pressure setpoint (bar) *)
    Temperature_SP : REAL := 200.0; (* Reaction temperature setpoint (°C) *)
    Pressure_Tolerance : REAL := 2.0; (* Allowable pressure deviation (bar) *)
    Temperature_Tolerance : REAL := 5.0; (* Allowable temperature deviation (°C) *)
    MinFlow : REAL := 1.0; (* Minimum flow rate for validation (L/min) *)
END_VAR

(* Helper method to validate inputs *)
METHOD PRIVATE ValidateInputs : BOOL
    ValidateInputs := IS_VALID_REAL(EthyleneFlow_PV) AND
                     IS_VALID_REAL(CatalystFlow_PV) AND
                     IS_VALID_REAL(Pressure_PV) AND
                     IS_VALID_REAL(Temperature_PV) AND
                     EthyleneFlow_PV >= 0.0 AND EthyleneFlow_PV <= 100.0 AND
                     CatalystFlow_PV >= 0.0 AND CatalystFlow_PV <= 50.0 AND
                     Pressure_PV >= 0.0 AND Pressure_PV <= 100.0 AND
                     Temperature_PV >= 0.0 AND Temperature_PV <= 300.0;
END_METHOD

(* Helper method to update temperature and pressure *)
METHOD PRIVATE UpdateTemperatureAndPressure
    VAR_INPUT
        TargetPressure : REAL; (* Desired pressure setpoint *)
        TargetTemperature : REAL; (* Desired temperature setpoint *)
    END_VAR
    (* Simplified control: assumes external PID controllers adjust actuators *)
    (* In practice, integrate with PID blocks or control logic *)
    IF ABS(Pressure_PV - TargetPressure) > Pressure_Tolerance OR
       ABS(Temperature_PV - TargetTemperature) > Temperature_Tolerance THEN
        FaultAlarm := TRUE;
        State := 4; (* Transition to Discharge for safety *)
    END_IF;
END_METHOD

(* Helper function to check REAL validity *)
FUNCTION IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_FUNCTION

(* Main control logic *)
(* Detect rising edge of StartBatch *)
StartTrigger(CLK := StartBatch);

(* State machine *)
CASE State OF
    0: (* Idle *)
        (* Reset outputs and flags *)
        EthyleneValveCmd := FALSE;
        CatalystValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        BatchComplete := FALSE;
        FaultAlarm := FALSE;
        PhaseTimer(IN := FALSE);
        IF StartTrigger.Q AND NOT EmergencyStop AND ValidateInputs() THEN
            State := 1; (* Start raw material preparation *)
        END_IF;
    
    1: (* Raw Material Preparation *)
        (* Load ethylene and catalyst *)
        EthyleneValveCmd := TRUE;
        CatalystValveCmd := TRUE;
        DischargeValveCmd := FALSE;
        IF EmergencyStop OR NOT ValidateInputs() OR
           NOT EthyleneValveStatus OR NOT CatalystValveStatus OR
           EthyleneFlow_PV < MinFlow OR CatalystFlow_PV < MinFlow THEN
            FaultAlarm := TRUE;
            State := 4; (* Transition to Discharge *)
        ELSIF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE);
            State := 2; (* Transition to Reaction *)
        ELSE
            PhaseTimer(IN := TRUE, PT := RawMatPrepDuration); (* 30s loading *)
        END_IF;
    
    2: (* Reaction *)
        (* Maintain reaction conditions *)
        EthyleneValveCmd := FALSE;
        CatalystValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        UpdateTemperatureAndPressure(Pressure_SP, Temperature_SP);
        IF EmergencyStop OR FaultAlarm THEN
            State := 4; (* Transition to Discharge *)
        ELSIF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE);
            State := 3; (* Transition to Cooling *)
        ELSE
            PhaseTimer(IN := TRUE, PT := ReactionDuration); (* 300s reaction *)
        END_IF;
    
    3: (* Cooling *)
        (* Cool reactor to safe temperature *)
        EthyleneValveCmd := FALSE;
        CatalystValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        UpdateTemperatureAndPressure(0.0, 50.0); (* Cool to 50°C *)
        IF EmergencyStop OR FaultAlarm THEN
            State := 4; (* Transition to Discharge *)
        ELSIF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE);
            State := 4; (* Transition to Discharge *)
        ELSE
            PhaseTimer(IN := TRUE, PT := CoolingDuration); (* 120s cooling *)
        END_IF;
    
    4: (* Discharge *)
        (* Discharge product and shutdown *)
        EthyleneValveCmd := FALSE;
        CatalystValveCmd := FALSE;
        DischargeValveCmd := TRUE;
        IF EmergencyStop OR NOT ValidateInputs() THEN
            FaultAlarm := TRUE;
        ELSIF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE);
            DischargeValveCmd := FALSE;
            BatchComplete := TRUE;
            State := 0; (* Return to Idle *)
        ELSE
            PhaseTimer(IN := TRUE, PT := DischargeDuration); (* 60s discharge *)
        END_IF;
    
    ELSE
        (* Invalid state: reset to Discharge *)
        EthyleneValveCmd := FALSE;
        CatalystValveCmd := FALSE;
        DischargeValveCmd := FALSE;
        FaultAlarm := TRUE;
        PhaseTimer(IN := FALSE);
        State := 4;
END_CASE;

END_PROGRAM
