FUNCTION_BLOCK FB_LauteringControl
VAR_INPUT
    Start_Lautering   : BOOL;
    Turbidity         : REAL;  // NTU
    LauterLevel       : REAL;  // mm or %
    TargetLevel       : REAL;
    SpargeTemp        : REAL;  // °C
    VolumeCollected   : REAL;  // Liters
    FlowRate          : REAL;  // L/min
    TimerEnable       : BOOL;
    MaxTime           : TIME;
END_VAR

VAR_OUTPUT
    RecirculationPump : BOOL;
    SpargeValve       : BOOL;
    DivertValveToKettle : BOOL;
    DivertValveToWaste  : BOOL;
    LauteringDone     : BOOL;
END_VAR

VAR
    Phase             : INT := 0;
    T_Lautering       : TON;
END_VAR

// ---------------- MAIN LOGIC ----------------

CASE Phase OF

    0: // Wait for start
        IF Start_Lautering THEN
            Phase := 1;
        END_IF

    1: // Start recirculation
        RecirculationPump := TRUE;
        IF Turbidity < 30.0 THEN
            Phase := 2;
        END_IF

    2: // Begin wort transfer to kettle
        DivertValveToKettle := TRUE;
        RecirculationPump := FALSE;
        Phase := 3;

    3: // Start sparging if level low
        IF LauterLevel < TargetLevel THEN
            SpargeValve := TRUE;
        ELSE
            SpargeValve := FALSE;
        END_IF

        // Turbidity recheck
        IF Turbidity > 40.0 THEN
            DivertValveToKettle := FALSE;
            DivertValveToWaste := TRUE;
        ELSE
            DivertValveToWaste := FALSE;
        END_IF

        // Start timer if not already running
        IF TimerEnable THEN
            T_Lautering(IN := TRUE, PT := MaxTime);
        END_IF

        // Check for end conditions
        IF VolumeCollected >= 1000.0 OR T_Lautering.Q THEN
            LauteringDone := TRUE;
            Phase := 4;
        END_IF

    4: // Stop all operations
        SpargeValve := FALSE;
        DivertValveToKettle := FALSE;
        RecirculationPump := FALSE;
        DivertValveToWaste := FALSE;

END_CASE
