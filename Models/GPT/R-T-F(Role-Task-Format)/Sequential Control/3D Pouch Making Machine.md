// IEC 61131-3 Structured Text Program for 3D Pouch Making Machine Start-up and Shutdown

VAR
    // === System Mode Flags ===
    Start_Command      : BOOL;
    Shutdown_Command   : BOOL;
    Machine_Ready      : BOOL := FALSE;
    Machine_Running    : BOOL := FALSE;
    Machine_ShuttingDown : BOOL := FALSE;

    // === Heating and Cooling Stations ===
    Heating_Stations   : ARRAY[1..8] OF BOOL;
    Cooling_Stations   : ARRAY[1..8] OF BOOL;
    Heating_Temp_OK    : ARRAY[1..8] OF BOOL;
    Cooling_Temp_OK    : ARRAY[1..8] OF BOOL;

    // === Cutters and Feeders ===
    Horizontal_Cutter  : BOOL := FALSE;
    Vertical_Cutter    : BOOL := FALSE;
    Feeder_1_Active    : BOOL := FALSE;
    Feeder_2_Active    : BOOL := FALSE;

    // === Tension Control ===
    Tension_OK         : BOOL := FALSE;

    // === Timers for startup sequencing ===
    t_Heat             : TON;
    t_Cool             : TON;
    t_Cutters          : TON;
    t_FeederDelay      : TON;

    // === Temperature and Delay Thresholds ===
    Heating_Target_Temp : REAL := 180.0;  // °C
    Cooling_Target_Temp : REAL := 60.0;   // °C
    Temp_Sensor         : ARRAY[1..8] OF REAL;
    Heat_Timeout        : TIME := T#60s;
    Cool_Timeout        : TIME := T#30s;
    Cutter_Delay        : TIME := T#5s;
    Feeder_Start_Delay  : TIME := T#3s;

    // === Step Tracking ===
    Startup_Step       : INT := 0;
    Shutdown_Step      : INT := 0;
END_VAR

// === START-UP SEQUENCE ===
IF Start_Command AND NOT Machine_Running THEN
    CASE Startup_Step OF

    0: // Begin heating
        FOR i := 1 TO 8 DO
            Heating_Stations[i] := TRUE;
        END_FOR;
        t_Heat(IN := TRUE, PT := Heat_Timeout);
        Startup_Step := 1;

    1: // Wait for heating to complete
        FOR i := 1 TO 8 DO
            Heating_Temp_OK[i] := Temp_Sensor[i] >= Heating_Target_Temp;
        END_FOR;

        IF t_Heat.Q AND (Heating_Temp_OK[1] AND Heating_Temp_OK[2] AND Heating_Temp_OK[3] AND
                         Heating_Temp_OK[4] AND Heating_Temp_OK[5] AND Heating_Temp_OK[6] AND
                         Heating_Temp_OK[7] AND Heating_Temp_OK[8]) THEN
            t_Heat(IN := FALSE);
            Startup_Step := 2;
        END_IF;

    2: // Begin cooling
        FOR i := 1 TO 8 DO
            Cooling_Stations[i] := TRUE;
        END_FOR;
        t_Cool(IN := TRUE, PT := Cool_Timeout);
        Startup_Step := 3;

    3: // Wait for cooling to stabilize
        FOR i := 1 TO 8 DO
            Cooling_Temp_OK[i] := Temp_Sensor[i] <= Cooling_Target_Temp;
        END_FOR;

        IF t_Cool.Q AND (Cooling_Temp_OK[1] AND Cooling_Temp_OK[2] AND Cooling_Temp_OK[3] AND
                         Cooling_Temp_OK[4] AND Cooling_Temp_OK[5] AND Cooling_Temp_OK[6] AND
                         Cooling_Temp_OK[7] AND Cooling_Temp_OK[8]) THEN
            t_Cool(IN := FALSE);
            Startup_Step := 4;
        END_IF;

    4: // Start feeder motors with delay for winding tension
        t_FeederDelay(IN := TRUE, PT := Feeder_Start_Delay);
        IF t_FeederDelay.Q THEN
            Feeder_1_Active := TRUE;
            Feeder_2_Active := TRUE;
            Tension_OK := TRUE;
            t_FeederDelay(IN := FALSE);
            Startup_Step := 5;
        END_IF;

    5: // Activate cutters after tension is established
        t_Cutters(IN := TRUE, PT := Cutter_Delay);
        IF t_Cutters.Q THEN
            Horizontal_Cutter := TRUE;
            Vertical_Cutter := TRUE;
            t_Cutters(IN := FALSE);
            Machine_Running := TRUE;
            Machine_Ready := TRUE;
        END_IF;

    END_CASE;
END_IF

// === SHUTDOWN SEQUENCE ===
IF Shutdown_Command AND Machine_Running THEN
    Machine_ShuttingDown := TRUE;
    CASE Shutdown_Step OF

    0: // Stop cutters immediately
        Horizontal_Cutter := FALSE;
        Vertical_Cutter := FALSE;
        Shutdown_Step := 1;

    1: // Stop feeders, release tension
        Feeder_1_Active := FALSE;
        Feeder_2_Active := FALSE;
        Tension_OK := FALSE;
        Shutdown_Step := 2;

    2: // Disable heating
        FOR i := 1 TO 8 DO
            Heating_Stations[i] := FALSE;
        END_FOR;
        Shutdown_Step := 3;

    3: // Continue cooling until safe temp
        FOR i := 1 TO 8 DO
            Cooling_Stations[i]
