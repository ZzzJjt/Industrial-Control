VAR
    (* Stage Control *)
    Step : INT := 0; // 0 = Idle, 1 = Polymerize, 2 = Decover, 3 = Dry
    Phase : INT := 0; // Substep within stage

    (* Inputs *)
    ReactorPressure : REAL;
    ReactorTemperature : REAL;

    (* Outputs *)
    Valve_Evacuate : BOOL := FALSE;
    Valve_Water : BOOL := FALSE;
    Valve_Surfactant : BOOL := FALSE;
    Heater : BOOL := FALSE;
    Agitator : BOOL := FALSE;
    Valve_Decover : BOOL := FALSE;
    Valve_Dry : BOOL := FALSE;

    (* Control Parameters *)
    Temp_SP : REAL := 57.5; // °C
    Temp_Low : REAL := 55.0;
    Temp_High : REAL := 60.0;
    Pressure_Threshold : REAL := 0.5; // bar
    Timeout_Evacuate : TIME := T#5m;
    Timeout_React : TIME := T#60m;
    Water_Add_Duration : TIME := T#1m;

    (* Internal *)
    Evac_Timer : TON;
    React_Timer : TON;
    Water_Timer : TON;
    PhaseDone : BOOL := FALSE;
    Start_Execution : BOOL := TRUE;
END_VAR

IF Phase = 1 THEN
    Valve_Evacuate := TRUE;
    Evac_Timer(IN := TRUE, PT := Timeout_Evacuate);

    IF (ReactorPressure <= Pressure_Threshold) OR (Evac_Timer.Q) THEN
        Valve_Evacuate := FALSE;
        Phase := 2;
    END_IF
END_IF

IF Phase = 4 THEN
    Heater := TRUE;
    Agitator := TRUE;
    React_Timer(IN := TRUE, PT := Timeout_React);

    IF (ReactorTemperature >= Temp_Low) AND (ReactorTemperature <= Temp_High) THEN
        // Continue
    ELSE
        // Optional: fault or delay handling
    END_IF

    IF (ReactorPressure <= Pressure_Threshold) OR (React_Timer.Q) THEN
        Heater := FALSE;
        Agitator := FALSE;
        Step := 2; // Proceed to Decover
        Phase := 0;
    END_IF
END_IF
