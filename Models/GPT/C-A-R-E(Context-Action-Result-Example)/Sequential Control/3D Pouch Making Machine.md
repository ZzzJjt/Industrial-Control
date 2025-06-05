VAR
    MachineState        : INT := 0; // 0=IDLE, 1=STARTUP, 2=RUNNING, 3=SHUTDOWN
    HeaterReady         : BOOL := FALSE;
    CoolerReady         : BOOL := FALSE;
    FeederReady         : BOOL := FALSE;
    CutterReady         : BOOL := FALSE;

    HeaterTemp          : ARRAY[1..8] OF REAL;
    CoolerTemp          : ARRAY[1..8] OF REAL;
    rTension            : REAL;
    rTensionSetpoint    : REAL := 50.0;

    HeatersOn           : ARRAY[1..8] OF BOOL;
    CoolersOn           : ARRAY[1..8] OF BOOL;
    FeedersOn           : ARRAY[1..2] OF BOOL;
    CuttersOn           : STRUCT
                            Horizontal : BOOL;
                            Vertical   : BOOL;
                         END_STRUCT;

    TimerCoolDown       : TON;
    tCoolTime           : TIME := T#5m;
END_VAR

IF MachineState = 1 THEN // STARTUP
    // Step 1: Enable all heaters
    FOR i := 1 TO 8 DO
        HeatersOn[i] := TRUE;
    END_FOR

    // Check heater temps
    HeaterReady := TRUE;
    FOR i := 1 TO 8 DO
        IF HeaterTemp[i] < 180.0 THEN
            HeaterReady := FALSE;
        END_IF
    END_FOR

    // Step 2: Enable coolers after heaters ready
    IF HeaterReady THEN
        FOR i := 1 TO 8 DO
            CoolersOn[i] := TRUE;
        END_FOR
        CoolerReady := TRUE; // Could include cooling temp checks if needed
    END_IF

    // Step 3: Activate feeder units when heating & cooling are ready
    IF HeaterReady AND CoolerReady THEN
        FOR i := 1 TO 2 DO
            FeedersOn[i] := TRUE;
        END_FOR
        // Tension check
        IF rTension >= rTensionSetpoint THEN
            FeederReady := TRUE;
        END_IF
    END_IF

    // Step 4: Activate cutters after tension stabilizes
    IF FeederReady THEN
        CuttersOn.Horizontal := TRUE;
        CuttersOn.Vertical := TRUE;
        CutterReady := TRUE;
    END_IF

    // Final transition to RUNNING
    IF CutterReady THEN
        MachineState := 2; // RUNNING
    END_IF
END_IF
