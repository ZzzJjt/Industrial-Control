(* Program: Pneumatic System Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Maintains flow rate at 50 SLPM and pressure between 5.5–6.0 bar *)
(* Executes on 100 ms cycle with safety checks for faults *)
PROGRAM PRG_PneumaticControl
VAR
    (* Inputs *)
    FlowInput : REAL;                 (* Current airflow rate, SLPM *)
    PressureInput : REAL;             (* Current system pressure, bar *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    FlowValveOutput : BOOL;           (* TRUE to open flow valve *)
    PressureReliefValve : BOOL;       (* TRUE to activate relief valve *)
    FlowError : BOOL;                 (* TRUE if flow deviation > 5.0 SLPM *)
    PressureError : BOOL;             (* TRUE if pressure outside 5.5–6.0 bar *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency stop *)
    
    (* Internal Constants *)
    FlowSetpoint : REAL := 50.0;      (* Target flow rate, 50 SLPM *)
    MinPressure : REAL := 5.5;        (* Minimum safe pressure, bar *)
    MaxPressure : REAL := 6.0;        (* Maximum safe pressure, bar *)
    FlowTolerance : REAL := 5.0;      (* Flow deviation tolerance, SLPM *)
    
    (* Internal Variables *)
    FlowDeviation : REAL;             (* Absolute difference between FlowInput and FlowSetpoint *)
    ValidFlow : BOOL;                 (* TRUE if FlowInput is numerically valid *)
    ValidPressure : BOOL;             (* TRUE if PressureInput is within bounds *)
END_VAR

(* Initialize outputs *)
FlowValveOutput := FALSE;             (* Flow valve closed *)
PressureReliefValve := FALSE;         (* Relief valve closed *)
FlowError := FALSE;                   (* No initial flow error *)
PressureError := FALSE;               (* No initial pressure error *)
AlarmActive := FALSE;                 (* No initial alarm *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt system and activate alarm *)
    FlowValveOutput := FALSE;         (* Close flow valve *)
    PressureReliefValve := FALSE;     (* Close relief valve *)
    FlowError := FALSE;               (* Clear flow error *)
    PressureError := FALSE;           (* Clear pressure error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Validate sensor inputs *)
(* Check for numerical stability to prevent overflow *)
ValidFlow := ABS(FlowInput) <= 1.0E10;
ValidPressure := ABS(PressureInput) <= 1.0E10;

IF NOT ValidFlow OR NOT ValidPressure THEN
    (* Invalid sensor readings: Stop system *)
    FlowValveOutput := FALSE;         (* Close flow valve *)
    PressureReliefValve := FALSE;     (* Close relief valve *)
    FlowError := TRUE;                (* Flag flow error *)
    PressureError := TRUE;            (* Flag pressure error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
END_IF;

(* Check pressure safety *)
(* Pressure outside 5.5–6.0 bar indicates unsafe condition *)
IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
    (* Unsafe pressure: Activate relief and stop flow *)
    PressureError := TRUE;            (* Flag pressure error *)
    PressureReliefValve := TRUE;      (* Open relief valve *)
    FlowValveOutput := FALSE;         (* Close flow valve *)
    FlowError := FALSE;               (* Clear flow error *)
    AlarmActive := TRUE;              (* Activate alarm *)
    RETURN;
ELSE
    (* Safe pressure: Proceed with control *)
    PressureError := FALSE;           (* Clear pressure error *)
    PressureReliefValve := FALSE;     (* Close relief valve *)
    AlarmActive := FALSE;             (* Clear alarm unless flow error *)
END_IF;

(* Flow control logic *)
(* Maintain flow at 50 SLPM *)
IF FlowInput < FlowSetpoint THEN
    (* Flow below setpoint: Open valve *)
    FlowValveOutput := TRUE;
ELSE
    (* Flow at or above setpoint: Close valve *)
    FlowValveOutput := FALSE;
END_IF;

(* Flow deviation check *)
(* Flag error if deviation exceeds 5.0 SLPM *)
FlowDeviation := ABS(FlowInput - FlowSetpoint);
IF FlowDeviation > FlowTolerance THEN
    FlowError := TRUE;                (* Flag flow error *)
    AlarmActive := TRUE;              (* Activate alarm *)
ELSE
    FlowError := FALSE;               (* Clear flow error *)
    (* AlarmActive already cleared unless pressure error *)
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* All outputs off if fault or emergency *)
IF EmergencyStop OR FlowError OR PressureError THEN
    FlowValveOutput := FALSE;         (* Ensure flow valve closed on fault *)
END_IF;

END_PROGRAM
