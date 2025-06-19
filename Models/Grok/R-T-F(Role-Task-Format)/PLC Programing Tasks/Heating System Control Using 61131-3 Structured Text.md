PROGRAM HeatingSystemControl
VAR
    (* Inputs *)
    TempSensor1 : REAL; (* Temperature sensor 1 reading (°C) *)
    TempSensor2 : REAL; (* Temperature sensor 2 reading (°C) *)
    TempSensor3 : REAL; (* Temperature sensor 3 reading (°C) *)
    
    (* Outputs *)
    HeatingOn : BOOL := FALSE; (* TRUE to activate heater *)
    SensorFault : BOOL := FALSE; (* TRUE if any sensor is out of bounds *)
    
    (* Internal variables *)
    AvgTemp : REAL; (* Average temperature of the three sensors *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid sensor reading *)
END_VAR

(* Initialize outputs *)
HeatingOn := FALSE;
SensorFault := FALSE;
ErrorCode := 0;

(* Validate sensor readings *)
IF NOT IS_VALID_REAL(TempSensor1) OR 
   NOT IS_VALID_REAL(TempSensor2) OR 
   NOT IS_VALID_REAL(TempSensor3) THEN
    (* Non-finite sensor reading (NaN or infinity) *)
    SensorFault := TRUE;
    HeatingOn := FALSE;
    ErrorCode := 1;
    RETURN;
END_IF;

(* Check for sensor faults: readings outside 10°C to 30°C *)
IF (TempSensor1 < 10.0 OR TempSensor1 > 30.0) OR
   (TempSensor2 < 10.0 OR TempSensor2 > 30.0) OR
   (TempSensor3 < 10.0 OR TempSensor3 > 30.0) THEN
    (* Sensor fault detected: disable heater *)
    SensorFault := TRUE;
    HeatingOn := FALSE;
    ErrorCode := 1;
ELSE
    (* No sensor faults: proceed with control logic *)
    SensorFault := FALSE;
    ErrorCode := 0;
    
    (* Compute average temperature *)
    AvgTemp := (TempSensor1 + TempSensor2 + TempSensor3) / 3.0;
    
    (* Validate average temperature *)
    IF NOT IS_VALID_REAL(AvgTemp) THEN
        SensorFault := TRUE;
        HeatingOn := FALSE;
        ErrorCode := 1;
        RETURN;
    END_IF;
    
    (* Hysteresis control for temperature regulation *)
    IF NOT HeatingOn AND AvgTemp < 20.0 THEN
        (* Turn heater on if average temperature drops below 20°C *)
        HeatingOn := TRUE;
    ELSIF HeatingOn AND AvgTemp > 22.0 THEN
        (* Turn heater off if average temperature exceeds 22°C *)
        HeatingOn := FALSE;
    END_IF;
END_IF;

(* Helper function to check if a REAL value is valid *)
FUNCTION IS_VALID_REAL : BOOL
VAR_INPUT
    Value : REAL;
END_VAR
IS_VALID_REAL := NOT (Value = 0.0 / 0.0) AND NOT (Value = 1.0 / 0.0);
END_FUNCTION

END_PROGRAM
