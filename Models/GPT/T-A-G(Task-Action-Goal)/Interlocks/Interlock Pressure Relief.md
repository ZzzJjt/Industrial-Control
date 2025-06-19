(* Overfill and Overpressure Protection System *)
PROGRAM Overfill_Overpressure_Interlock
VAR
LevelSensor : REAL; // Vessel level in %
PressureSensor : REAL; // Vessel pressure in bar
INLET_VALVE : BOOL := TRUE; // TRUE = Open, FALSE = Closed
RELIEF_VALVE : BOOL := FALSE; // TRUE = Open, FALSE = Closed
SensorFault_Level : BOOL := FALSE;
SensorFault_Pressure : BOOL := FALSE;
ValveFault_Inlet : BOOL := FALSE;
ValveFault_Relief : BOOL := FALSE;
HighLevelTrip : BOOL := FALSE;
HighPressureTrip : BOOL := FALSE;
ManualReset : BOOL := FALSE;
END_VAR

// Overfill Protection Logic
IF SensorFault_Level OR ValveFault_Inlet THEN
INLET_VALVE := FALSE; // Fail-safe close
ELSE
IF HighLevelTrip THEN
IF LevelSensor < 80.0 AND ManualReset THEN
HighLevelTrip := FALSE;
ManualReset := FALSE; // Reset latch manually
END_IF
ELSE
IF LevelSensor > 90.0 THEN
HighLevelTrip := TRUE;
END_IF
END_IF;

IF HighLevelTrip THEN
    INLET_VALVE := FALSE;
ELSE
    INLET_VALVE := TRUE;
END_IF

END_IF

// Overpressure Protection Logic
IF SensorFault_Pressure OR ValveFault_Relief THEN
RELIEF_VALVE := TRUE; // Fail-safe open
ELSE
IF HighPressureTrip THEN
IF PressureSensor < 12.0 AND ManualReset THEN
HighPressureTrip := FALSE;
ManualReset := FALSE; // Reset latch manually
END_IF
ELSE
IF PressureSensor > 15.0 THEN
HighPressureTrip := TRUE;
END_IF
END_IF;

IF HighPressureTrip THEN
    RELIEF_VALVE := TRUE;
ELSE
    RELIEF_VALVE := FALSE;
END_IF

END_IF

END_PROGRAM
