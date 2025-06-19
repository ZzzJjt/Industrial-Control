VAR
    Phase : INT := 0; // Batch phase control

    // Equipment control
    Reactor_Vacuum : BOOL := FALSE;
    Water_Valve : BOOL := FALSE;
    Surfactant_Valve : BOOL := FALSE;
    VCM_Valve : BOOL := FALSE;
    Catalyst_Valve : BOOL := FALSE;
    Heater : BOOL := FALSE;
    Dryer : BOOL := FALSE;
    Pressure_OK : BOOL := FALSE;
    Dryness_OK : BOOL := FALSE;

    // Process conditions
    Temp_PV : REAL;
    Temp_SP : REAL := 57.5;
    Temp_Tolerance : REAL := 2.5;
    Pressure_PV : REAL;
    Pressure_Initial : REAL;
    Pressure_Final : REAL;
    PressureDrop_OK : BOOL := FALSE;

    // Timers
    StepTimer : TON;
    StepTrig : BOOL := FALSE;
    TimerDone : BOOL;

    // Output status
    Batch_Complete : BOOL := FALSE;
END_VAR

CASE Phase OF

    0: // Evacuate reactor
        Reactor_Vacuum := TRUE;
        StartTimer(T#2m);
        IF TimerDone THEN
            Reactor_Vacuum := FALSE;
            Phase := 1;
        END_IF

    1: // Add demineralized water
        AddDemineralizedWater(1000.0); // Liters
        IF Water_Added THEN
            Phase := 2;
        END_IF

    2: // Add surfactants
        AddSurfactants(50.0); // Liters
        IF Surfactants_Added THEN
            Phase := 3;
        END_IF

    3: // Add VCM and catalyst
        AddVCM(500.0);
        AddCatalyst(5.0);
        IF VCM_Added AND Catalyst_Added THEN
            Phase := 4;
        END_IF

    4: // Maintain temperature and monitor pressure drop
        Heater := TRUE;
        IF WithinTemperature(Temp_PV, Temp_SP, Temp_Tolerance) THEN
            Pressure_Initial := Pressure_PV;
            StartTimer(T#30m);
            Phase := 5;
        END_IF

    5: // Reaction time + monitor pressure drop
        IF TimerDone THEN
            Pressure_Final := Pressure_PV;
            IF (Pressure_Initial - Pressure_Final) > 0.3 THEN
                PressureDrop_OK := TRUE;
            END_IF
            Heater := FALSE;
            Phase := 6;
        END_IF

    6: // Decover phase
        Decover();
        Phase := 7;

    7: // Dry phase
        Dryer := TRUE;
        StartTimer(T#15m);
        IF TimerDone THEN
            Dryer := FALSE;
            Dryness_OK := TRUE;
            Phase := 8;
        END_IF

    8: // Batch complete
        Batch_Complete := TRUE;

END_CASE

METHOD StartTimer
VAR_INPUT Duration : TIME; END_VAR
StepTrig := TRUE;
StepTimer(IN := StepTrig, PT := Duration);
TimerDone := StepTimer.Q;

METHOD AddDemineralizedWater
VAR_INPUT Volume_L : REAL; END_VAR
Water_Valve := TRUE;
Water_Added := TRUE; // Simulated completion

METHOD AddSurfactants
VAR_INPUT Volume_L : REAL; END_VAR
Surfactant_Valve := TRUE;
Surfactants_Added := TRUE;

METHOD AddVCM
VAR_INPUT Volume_L : REAL; END_VAR
VCM_Valve := TRUE;
VCM_Added := TRUE;

METHOD AddCatalyst
VAR_INPUT Volume_L : REAL; END_VAR
Catalyst_Valve := TRUE;
Catalyst_Added := TRUE;

METHOD Decover
// Logic to safely depressurize and vent

METHOD WithinTemperature : BOOL
VAR_INPUT Actual, Setpoint, Tol : REAL;
WithinTemperature := ABS(Actual - Setpoint) <= Tol;
END_METHOD

