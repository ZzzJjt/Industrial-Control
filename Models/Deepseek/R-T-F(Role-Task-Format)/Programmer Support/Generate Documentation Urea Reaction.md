// Initialize timers
Timer_Load(IN := FALSE);
Timer_Reaction(IN := FALSE);
Timer_Shutdown(IN := FALSE);

// Step 1: Raw Material Loading
IF RawMaterialReady AND NOT Step1_Complete THEN
    Timer_Load(IN := TRUE);
    Timer_Load(PT := LoadingDuration);
    LoadValveOpen := TRUE;
END_IF;

IF Timer_Load.Q THEN
    Timer_Load(IN := FALSE);
    LoadValveOpen := FALSE;
    Step1_Complete := TRUE;
END_IF;

// Step 2: Reaction Control
IF Step1_Complete AND NOT Step2_Complete THEN
    Timer_Reaction(IN := TRUE);
    Timer_Reaction(PT := ReactionDuration);
    ReactionValveOpen := TRUE;
END_IF;

IF Timer_Reaction.Q THEN
    Timer_Reaction(IN := FALSE);
    ReactionValveOpen := FALSE;
    Step2_Complete := TRUE;
END_IF;

// Final Step: Automatic Shutdown
IF Step2_Complete AND NOT ReactionComplete THEN
    Timer_Shutdown(IN := TRUE);
    Timer_Shutdown(PT := ShutdownDelay);
END_IF;

IF Timer_Shutdown.Q THEN
    Timer_Shutdown(IN := FALSE);
    ShutdownValveOpen := TRUE;
    ReactionComplete := TRUE;
END_IF;

PROGRAM UreaReactionControl
VAR
    // Inputs
    RawMaterialReady : BOOL; // Indicates if raw materials are ready for loading.
    TemperatureSensor : REAL; // Current temperature reading from the sensor.
    PressureSensor : REAL; // Current pressure reading from the sensor.
    EmergencyStopSignal : BOOL; // Emergency stop signal input.

    // Outputs
    LoadValveOpen : BOOL; // Control signal to open the load valve.
    ReactionValveOpen : BOOL; // Control signal to open the reaction valve.
    ShutdownValveOpen : BOOL; // Control signal to open the shutdown valve.

    // Internal Flags
    Step1_Complete : BOOL; // Flag indicating completion of raw material loading.
    Step2_Complete : BOOL; // Flag indicating completion of reaction control.
    ReactionComplete : BOOL; // Flag indicating successful completion of the reaction.

    // Configurable Parameters
    TargetTemperature : REAL := 180.0; // Target temperature for the reaction.
    TargetPressure : REAL := 150.0; // Target pressure for the reaction.
    LoadingDuration : TIME := T#10s; // Duration for which the load valve remains open.
    ReactionDuration : TIME := T#30m; // Duration for which the reaction valve remains open.
    ShutdownDelay : TIME := T#5s; // Delay before opening the shutdown valve after reaction completes.

    // Timers
    Timer_Load : TON; // Timer for loading duration.
    Timer_Reaction : TON; // Timer for reaction duration.
    Timer_Shutdown : TON; // Timer for shutdown delay.
END_VAR

// Initialize timers
Timer_Load(IN := FALSE);
Timer_Reaction(IN := FALSE);
Timer_Shutdown(IN := FALSE);

// Emergency stop condition
IF EmergencyStopSignal THEN
    LoadValveOpen := FALSE;
    ReactionValveOpen := FALSE;
    ShutdownValveOpen := FALSE;
    RETURN; // Exit the program
END_IF;

// Overtemperature protection
IF TemperatureSensor > 200.0 THEN // Threshold example
    LoadValveOpen := FALSE;
    ReactionValveOpen := FALSE;
    ShutdownValveOpen := FALSE;
    RETURN; // Exit the program
END_IF;

// Overpressure protection
IF PressureSensor > 170.0 THEN // Threshold example
    LoadValveOpen := FALSE;
    ReactionValveOpen := FALSE;
    ShutdownValveOpen := FALSE;
    RETURN; // Exit the program
END_IF;

// Step 1: Raw Material Loading
IF RawMaterialReady AND NOT Step1_Complete THEN
    Timer_Load(IN := TRUE);
    Timer_Load(PT := LoadingDuration);
    LoadValveOpen := TRUE;
END_IF;

IF Timer_Load.Q THEN
    Timer_Load(IN := FALSE);
    LoadValveOpen := FALSE;
    Step1_Complete := TRUE;
END_IF;

// Step 2: Reaction Control
IF Step1_Complete AND NOT Step2_Complete THEN
    Timer_Reaction(IN := TRUE);
    Timer_Reaction(PT := ReactionDuration);
    ReactionValveOpen := TRUE;
END_IF;

IF Timer_Reaction.Q THEN
    Timer_Reaction(IN := FALSE);
    ReactionValveOpen := FALSE;
    Step2_Complete := TRUE;
END_IF;

// Final Step: Automatic Shutdown
IF Step2_Complete AND NOT ReactionComplete THEN
    Timer_Shutdown(IN := TRUE);
    Timer_Shutdown(PT := ShutdownDelay);
END_IF;

IF Timer_Shutdown.Q THEN
    Timer_Shutdown(IN := FALSE);
    ShutdownValveOpen := TRUE;
    ReactionComplete := TRUE;
END_IF;
