VAR
    Phase : INT := 0; // 0 = Idle, 1 = StartHeating, 2 = HoldPressure, 3 = StartCooling
    Temp  : REAL;     // Reactor temperature (°C)
    Temp_SP : REAL := 180.0;
    Temp_Cool_SP : REAL := 80.0;

    Pressure : REAL;   // Reactor pressure (bar)
    Pressure_SP : REAL := 140.0;

    Timer : TON;
    TimerStarted : BOOL := FALSE;

    Heater : BOOL;
    Cooler : BOOL;
    PressureControl : BOOL;

    PhaseDone : BOOL := FALSE;
    ReactionDone : BOOL := FALSE;
END_VAR

CASE Phase OF

// 1. Start Heating Phase
1:
    Heater := TRUE;
    Cooler := FALSE;
    IF Temp >= Temp_SP THEN
        Heater := FALSE;
        Phase := 2;
    END_IF

// 2. Hold Pressure Phase (30 minutes)
2:
    PressureControl := TRUE;
    IF NOT TimerStarted THEN
        Timer(IN := TRUE, PT := T#30m);
        TimerStarted := TRUE;
    END_IF

    IF Timer.Q AND (Pressure >= Pressure_SP - 2.0) THEN
        Timer(IN := FALSE);
        TimerStarted := FALSE;
        PressureControl := FALSE;
        Phase := 3;
    END_IF

// 3. Start Cooling Phase
3:
    Cooler := TRUE;
    IF Temp <= Temp_Cool_SP THEN
        Cooler := FALSE;
        Phase := 0;
        ReactionDone := TRUE;
    END_IF

END_CASE
