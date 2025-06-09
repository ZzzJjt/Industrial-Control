VAR
    // Machine states
    MachineState : INT := 0; // 0=Idle, 1=StartUp, 2=Running, 3=ShuttingDown

    // Heating & cooling status
    Heater_Temp : ARRAY[1..8] OF REAL;
    Heater_Ready : BOOL;
    Heater_Setpoint : REAL := 180.0; // °C
    Cooling_Ready : BOOL;

    // Timers
    T_HeatReady : TON;
    T_CoolWait : TON;
    Tension_RampDown_Timer : TON;

    // Cutters and feeders
    HorizontalCutter_On : BOOL := FALSE;
    VerticalCutter_On : BOOL := FALSE;
    FeederA_Speed : REAL := 0.0;
    FeederB_Speed : REAL := 0.0;

    // Tension control
    Tension_Target : REAL := 50.0;
    Tension_Current : REAL;
    Tension_OK : BOOL;

    // Internal flags
    All_Heaters_Ready : BOOL;
    Shutdown_Complete : BOOL := FALSE;
END_VAR

// ------------------------
// Start-Up Sequence Logic
// ------------------------
IF MachineState = 1 THEN // Start-up

    // 1. Activate all heating and cooling stations
    FOR i := 1 TO 8 DO
        Heater_Temp[i] := Heater_Temp[i] + 1.0; // simulate temp rising
    END_FOR;

    // 2. Wait for all heaters to reach the setpoint
    All_Heaters_Ready := TRUE;
    FOR i := 1 TO 8 DO
        IF Heater_Temp[i] < Heater_Setpoint THEN
            All_Heaters_Ready := FALSE;
        END_IF;
    END_FOR;

    T_HeatReady(IN := All_Heaters_Ready, PT := T#10s); // delay for thermal stability
    Heater_Ready := T_HeatReady.Q;

    // 3. When heaters are ready, enable feeders and regulate tension
    IF Heater_Ready THEN
        FeederA_Speed := 50.0;
        FeederB_Speed := 50.0;
        Tension_Current := Tension_Target;
        Tension_OK := TRUE;
    END_IF;

    // 4. Enable cutters when feeders and tension are stabilized
    IF Tension_OK THEN
        HorizontalCutter_On := TRUE;
        VerticalCutter_On := TRUE;
        MachineState := 2; // Transition to RUNNING
    END_IF;
END_IF

// ------------------------
// Shutdown Sequence Logic
// ------------------------
IF MachineState = 3 THEN // Shutdown

    // 1. Stop cutters and feeders
    HorizontalCutter_On := FALSE;
    VerticalCutter_On := FALSE;
    FeederA_Speed := 0.0;
    FeederB_Speed := 0.0;

    // 2. Gradually release tension
    Tension_RampDown_Timer(IN := TRUE, PT := T#5s);
    IF Tension_RampDown_Timer.Q THEN
        Tension_Current := 0.0;
    ELSE
        Tension_Current := Tension_Target * (1.0 - (Tension_RampDown_Timer.ET / Tension_RampDown_Timer.PT));
    END_IF;

    // 3. Wait for cooling to stabilize before shutdown
    T_CoolWait(IN := TRUE, PT := T#15s);
    Cooling_Ready := T_CoolWait.Q;

    IF Cooling_Ready AND (Tension_Current <= 0.1) THEN
        Shutdown_Complete := TRUE;
        MachineState := 0; // Return to Idle
    END_IF;
END_IF
