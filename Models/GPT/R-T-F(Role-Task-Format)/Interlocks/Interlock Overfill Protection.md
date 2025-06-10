(* === Variable Declarations === *)
VAR
    LevelSensor      : REAL;   (* Vessel liquid level in % *)
    HighLevelSetpoint: REAL := 90.0;    (* Overfill threshold (%) *)
    ResetThreshold   : REAL := 80.0;    (* Level must drop below this to reset *)

    INLET_VALVE      : BOOL;   (* Output: TRUE = Valve Open, FALSE = Closed *)

    SensorFault      : BOOL;   (* TRUE if sensor is faulty or reading invalid *)
    ValveFault       : BOOL;   (* TRUE if valve fails to operate properly *)
    InterlockActive  : BOOL;   (* Latches TRUE if overfill condition or fault detected *)
    RESET            : BOOL;   (* Manual reset signal *)
END_VAR

(* === Overfill Detection and Fault Response Logic === *)

(* Trigger interlock if level exceeds high setpoint or faults are detected *)
IF (LevelSensor > HighLevelSetpoint) OR SensorFault OR ValveFault THEN
    InterlockActive := TRUE;
END_IF;

(* Manual reset allowed only if: 
   - level has dropped below ResetThreshold
   - no active sensor or valve faults *)
IF RESET AND (LevelSensor < ResetThreshold) AND NOT SensorFault AND NOT ValveFault THEN
    InterlockActive := FALSE;
END_IF;

(* Control inlet valve: Fail-safe closed during interlock *)
IF InterlockActive THEN
    INLET_VALVE := FALSE;  (* CLOSE inlet valve *)
ELSE
    INLET_VALVE := TRUE;   (* OPEN inlet valve *)
END_IF;
