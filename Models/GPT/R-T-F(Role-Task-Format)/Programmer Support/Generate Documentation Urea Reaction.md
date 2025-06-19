VAR
    // Inputs
    Start_Command       : BOOL;
    Emergency_Stop      : BOOL;
    Sensor_Ammonia_OK   : BOOL;
    Sensor_CO2_OK       : BOOL;
    Temp_PV             : REAL;      // °C
    Pressure_PV         : REAL;      // bar
    CurrentTime         : TIME;

    // Outputs
    Valve_Ammonia       : BOOL := FALSE;
    Valve_CO2           : BOOL := FALSE;
    Valve_Discharge     : BOOL := FALSE;
    Alarm_Flag          : BOOL := FALSE;
    Batch_Complete      : BOOL := FALSE;

    // Internal flags
    Step_Loading        : BOOL := FALSE;
    Step_Reaction       : BOOL := FALSE;
    Step_Shutdown       : BOOL := FALSE;
    ReactionTimerStart  : TIME;
    ElapsedTime         : TIME;

    // Parameters
    Temp_SP             : REAL := 185.0;
    Pressure_SP         : REAL := 150.0;
    Tolerance_Temp      : REAL := 5.0;
    Tolerance_Pressure  : REAL := 5.0;
    ReactionDuration    : TIME := T#30m;
END_VAR

// Emergency Handling
IF Emergency_Stop THEN
    Valve_Ammonia := FALSE;
    Valve_CO2 := FALSE;
    Valve_Discharge := FALSE;
    Alarm_Flag := TRUE;
    Step_Loading := FALSE;
    Step_Reaction := FALSE;
    Step_Shutdown := FALSE;
    Batch_Complete := FALSE;
    RETURN;
END_IF

// Step 1: Raw Material Loading
IF Start_Command AND NOT Step_Loading AND NOT Step_Reaction AND NOT Step_Shutdown THEN
    IF Sensor_Ammonia_OK AND Sensor_CO2_OK THEN
        Valve_Ammonia := TRUE;
        Valve_CO2 := TRUE;
        Step_Loading := TRUE;
    END_IF
END_IF

// Transition to Reaction Phase
IF Step_Loading AND NOT Step_Reaction THEN
    Valve_Ammonia := FALSE;
    Valve_CO2 := FALSE;
    Step_Loading := FALSE;
    Step_Reaction := TRUE;
    ReactionTimerStart := CurrentTime;
END_IF

// Step 2: Reaction Control
IF Step_Reaction THEN
    ElapsedTime := CurrentTime - ReactionTimerStart;

    // Deviation Check
    IF ABS(Temp_PV - Temp_SP) > Tolerance_Temp OR
       ABS(Pressure_PV - Pressure_SP) > Tolerance_Pressure THEN
        Alarm_Flag := TRUE;
        Step_Reaction := FALSE;
        Step_Shutdown := TRUE;
    END_IF

    // Reaction complete
    IF ElapsedTime >= ReactionDuration THEN
        Step_Reaction := FALSE;
        Step_Shutdown := TRUE;
    END_IF
END_IF

// Step 3: Shutdown and Completion
IF Step_Shutdown THEN
    Valve_Discharge := TRUE;
    Batch_Complete := TRUE;
END_IF
