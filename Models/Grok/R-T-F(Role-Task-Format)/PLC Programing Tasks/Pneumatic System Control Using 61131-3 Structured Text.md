PROGRAM PneumaticSystemControl
VAR
    (* Inputs *)
    FlowInput : REAL; (* Actual flow rate in standard liters per minute (SLPM) *)
    PressureInput : REAL; (* Actual pressure in bar *)
    
    (* Constants *)
    FlowSetpoint : REAL := 50.0; (* Target flow rate (SLPM) *)
    MinPressure : REAL := 5.5; (* Minimum safe pressure (bar) *)
    MaxPressure : REAL := 6.0; (* Maximum safe pressure (bar) *)
    FlowErrorThreshold : REAL := 5.0; (* Maximum allowable flow deviation (SLPM) *)
    
    (* Outputs *)
    FlowValveOutput : BOOL := FALSE; (* TRUE to open flow valve *)
    PressureReliefValve : BOOL := FALSE; (* TRUE to open pressure relief valve *)
    FlowError : BOOL := FALSE; (* TRUE if flow deviation exceeds threshold *)
    PressureError : BOOL := FALSE; (* TRUE if pressure is out of safe range *)
    
    (* Internal variables *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid input *)
    FlowDeviation : REAL; (* Absolute deviation between actual and setpoint flow *)
END_VAR

(* Initialize outputs *)
FlowValveOutput := FALSE;
PressureReliefValve := FALSE;
FlowError := FALSE;
PressureError := FALSE;
ErrorCode := 0;

(* Validate inputs *)
IF NOT IS_VALID_REAL(FlowInput) OR NOT IS_VALID_REAL(PressureInput) THEN
    (* Non-finite input (NaN or infinity) *)
    FlowValveOutput := FALSE;
    PressureReliefValve := TRUE; (* Open relief valve for safety *)
    FlowError := TRUE;
    PressureError := TRUE;
    ErrorCode := 1;
    RETURN;
END_IF;

(* Pressure safety logic *)
IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
    (* Pressure out of safe range: activate relief valve and flag error *)
    PressureError := TRUE;
    PressureReliefValve := TRUE;
    FlowValveOutput := FALSE; (* Close flow valve to prevent further pressure issues *)
ELSE
    (* Pressure within safe range *)
    PressureError := FALSE;
    PressureReliefValve := FALSE;
    
    (* Flow control logic *)
    IF FlowInput < FlowSetpoint THEN
        (* Flow below setpoint: open valve to increase flow *)
        FlowValveOutput := TRUE;
    ELSE
        (* Flow at or above setpoint: close valve to maintain or reduce flow *)
        FlowValveOutput := FALSE;
    END_IF;
END_IF;

(* Flow deviation detection *)
FlowDeviation := ABS(FlowInput - FlowSetpoint);
IF FlowDeviation > FlowErrorThreshold THEN
    (* Flow deviation exceeds 5.0 SLPM: flag error *)
    FlowError := TRUE;
ELSE
    FlowError := FALSE;
END_IF;

(* Helper function to check if a REAL value is valid *)
FUNCTION IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_FUNCTION

END_PROGRAM
