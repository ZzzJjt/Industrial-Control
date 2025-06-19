VAR
    // Process Parameters
    Temp_Setpoint : REAL := 70.0;      // Target temperature in °C
    Mixing_Speed  : INT := 200;        // RPM
    Mixing_Time   : TIME := T#10m;     // Mixing duration

    // Inputs (from sensors/actuators)
    Temperature   : REAL;              // Actual vessel temperature
    Mixer_Speed   : INT;               // Actual RPM

    // Outputs
    Valve_Milk      : BOOL := FALSE;
    Valve_Water     : BOOL := FALSE;
    Valve_Sugar     : BOOL := FALSE;
    Valve_Cocoa     : BOOL := FALSE;
    Heater          : BOOL := FALSE;
    Mixer_Start     : BOOL := FALSE;

    // Control
    BatchStep       : INT := 0;         // 0=Idle, 1=Add, 2=Heat, 3=Mix, 4=Done
    Timer_Mixing    : TON;
    MixingStarted   : BOOL := FALSE;
    Batch_Complete  : BOOL := FALSE;
END_VAR
