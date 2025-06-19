PROGRAM UreaReactionControl
VAR
    // Inputs
    AmmoniaDetected, CO2Detected : BOOL;
    ReactorTemp, ReactorPressure : REAL;

    // Outputs
    AmmoniaValveOpen, CO2ValveOpen, ReactorHeaterOn : BOOL;
    PressureReliefValve, ReactionComplete : BOOL;

    // Parameters
    MinTemp, MaxTemp : REAL := 180.0, 200.0; // °C
    MinPressure, MaxPressure : REAL := 140.0, 160.0; // bar
    ReactionDuration : TIME := T#30m;

    // Internal
    Step : INT := 0;
    StartTime : TIME := CURRENT_TIME;
    OverheatDetected, OverpressureDetected : BOOL := FALSE;
END_VAR
