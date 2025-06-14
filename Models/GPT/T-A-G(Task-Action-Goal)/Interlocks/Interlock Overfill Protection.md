(* Overfill Protection Interlock for Process Vessel *)

FUNCTION_BLOCK FB_OverfillProtection
VAR_INPUT
LevelSensor       : REAL;   // Vessel level in %
SensorValid       : BOOL;   // TRUE if level sensor signal is valid
ValveOK           : BOOL;   // TRUE if valve actuator is working properly
ManualReset       : BOOL;   // Operator manual reset trigger
END_VAR

VAR_OUTPUT
INLET_VALVE       : BOOL;   // TRUE = open, FALSE = closed
Alarm_Overfill    : BOOL;   // TRUE if overfill condition is latched
Alarm_SensorFail  : BOOL;   // TRUE if sensor signal invalid
Alarm_ValveFail   : BOOL;   // TRUE if valve is faulty
END_VAR

VAR
ShutdownLatched   : BOOL := FALSE;
H_Level           : REAL := 90.0;  // High level shutdown threshold
R_Level           : REAL := 85.0;  // Reset level threshold
END_VAR

// Evaluate fault conditions
Alarm_SensorFail := NOT SensorValid;
Alarm_ValveFail := NOT ValveOK;

// Latching logic for shutdown
IF SensorValid AND (LevelSensor >= H_Level) THEN
ShutdownLatched := TRUE;
END_IF

// Allow reset if level is safe and manual reset is triggered
IF (LevelSensor < R_Level) AND ManualReset THEN
ShutdownLatched := FALSE;
END_IF

// Apply interlock logic
IF ShutdownLatched OR Alarm_SensorFail OR Alarm_ValveFail THEN
INLET_VALVE := FALSE;  // Force valve closed
ELSE
INLET_VALVE := TRUE;   // Allow valve to open under normal condition
END_IF

// Raise alarm if shutdown is active
Alarm_Overfill := ShutdownLatched;

END_FUNCTION_BLOCK
