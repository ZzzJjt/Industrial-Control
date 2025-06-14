PROGRAM PouchMakingMachineControl
VAR
    // Inputs
    StartButton : BOOL; // Start button input
    StopButton : BOOL;  // Stop button input
    EmergencyStop : BOOL; // Emergency stop input

    // Outputs
    HeatingStations : ARRAY[1..8] OF BOOL; // Heating station outputs
    CoolingStations : ARRAY[1..8] OF BOOL; // Cooling station outputs
    FeederUnit1_Speed : INT; // Speed control for Feeder Unit 1
    FeederUnit2_Speed : INT; // Speed control for Feeder Unit 2
    HorizontalCutter_Active : BOOL; // Horizontal cutter output
    VerticalCutter_Active : BOOL; // Vertical cutter output

    // Internal Variables
    StartupState : INT := 0; // State variable for startup sequence
    ShutdownState : INT := 0; // State variable for shutdown sequence
    Running : BOOL := FALSE; // Machine running status
    EmergencyStopped : BOOL := FALSE; // Emergency stop status
    TensionThreshold1 : INT := 50; // Tension threshold for Feeder Unit 1
    TensionThreshold2 : INT := 50; // Tension threshold for Feeder Unit 2
    MaterialSpeed : INT := 50; // Material speed setpoint
    TimerStartup : TON; // Timer for startup sequence
    TimerShutdown : TON; // Timer for shutdown sequence
END_VAR

// Initialize variables
HeatingStations := [FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE];
CoolingStations := [FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE];
FeederUnit1_Speed := 0;
FeederUnit2_Speed := 0;
HorizontalCutter_Active := FALSE;
VerticalCutter_Active := FALSE;

// Main control loop
IF EmergencyStop THEN
    EmergencyStopped := TRUE;
    Running := FALSE;
    ShutdownState := 1; // Initiate shutdown sequence
ELSE
    IF StartButton AND NOT Running AND NOT EmergencyStopped THEN
        Running := TRUE;
        StartupState := 1; // Initiate startup sequence
    ELSIF StopButton AND Running THEN
        Running := FALSE;
        ShutdownState := 1; // Initiate shutdown sequence
    END_IF;
END_IF;

// Startup Sequence
IF Running AND StartupState > 0 THEN
    CASE StartupState OF
        1: // Power on Heating Stations sequentially
            TimerStartup(IN := TRUE, PT := T#1s);
            IF TimerStartup.Q THEN
                TimerStartup(IN := FALSE);
                HeatingStations[StartupState] := TRUE;
                StartupState := StartupState + 1;
            END_IF;
        2 TO 8: // Continue powering on Heating Stations
            TimerStartup(IN := TRUE, PT := T#1s);
            IF TimerStartup.Q THEN
                TimerStartup(IN := FALSE);
                HeatingStations[StartupState] := TRUE;
                StartupState := StartupState + 1;
            END_IF;
        9: // Power on Cooling Stations sequentially
            TimerStartup(IN := TRUE, PT := T#1s);
            IF TimerStartup.Q THEN
                TimerStartup(IN := FALSE);
                CoolingStations[StartupState - 8] := TRUE;
                StartupState := StartupState + 1;
            END_IF;
        10 TO 16: // Continue powering on Cooling Stations
            TimerStartup(IN := TRUE, PT := T#1s);
            IF TimerStartup.Q THEN
                TimerStartup(IN := FALSE);
                CoolingStations[StartupState - 8] := TRUE;
                StartupState := StartupState + 1;
            END_IF;
        17: // Set Feeder Unit speeds
            FeederUnit1_Speed := MaterialSpeed;
            FeederUnit2_Speed := MaterialSpeed;
            StartupState := StartupState + 1;
        18: // Activate Cutters
            HorizontalCutter_Active := TRUE;
            VerticalCutter_Active := TRUE;
            StartupState := 0; // Startup complete
        ELSE
            StartupState := 0; // Reset startup state
    END_CASE;
END_IF;

// Shutdown Sequence
IF NOT Running AND ShutdownState > 0 THEN
    CASE ShutdownState OF
        1: // Deactivate Cutters
            HorizontalCutter_Active := FALSE;
            VerticalCutter_Active := FALSE;
            ShutdownState := ShutdownState + 1;
        2: // Set Feeder Unit speeds to zero
            FeederUnit1_Speed := 0;
            FeederUnit2_Speed := 0;
            ShutdownState := ShutdownState + 1;
        3 TO 8: // Power off Cooling Stations sequentially
            TimerShutdown(IN := TRUE, PT := T#1s);
            IF TimerShutdown.Q THEN
                TimerShutdown(IN := FALSE);
                CoolingStations[9 - ShutdownState] := FALSE;
                ShutdownState := ShutdownState + 1;
            END_IF;
        9 TO 14: // Continue powering off Cooling Stations
            TimerShutdown(IN := TRUE, PT := T#1s);
            IF TimerShutdown.Q THEN
                TimerShutdown(IN := FALSE);
                CoolingStations[9 - ShutdownState] := FALSE;
                ShutdownState := ShutdownState + 1;
            END_IF;
        15: // Power off Heating Stations sequentially
            TimerShutdown(IN := TRUE, PT := T#1s);
            IF TimerShutdown.Q THEN
                TimerShutdown(IN := FALSE);
                HeatingStations[16 - ShutdownState] := FALSE;
                ShutdownState := ShutdownState + 1;
            END_IF;
        16 TO 21: // Continue powering off Heating Stations
            TimerShutdown(IN := TRUE, PT := T#1s);
            IF TimerShutdown.Q THEN
                TimerShutdown(IN := FALSE);
                HeatingStations[16 - ShutdownState] := FALSE;
                ShutdownState := ShutdownState + 1;
            END_IF;
        22: // Complete shutdown
            ShutdownState := 0; // Shutdown complete
        ELSE
            ShutdownState := 0; // Reset shutdown state
    END_CASE;
END_IF;

// Emergency Stop Handling
IF EmergencyStopped THEN
    // Ensure all components are deactivated
    HeatingStations := [FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE];
    CoolingStations := [FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE];
    FeederUnit1_Speed := 0;
    FeederUnit2_Speed := 0;
    HorizontalCutter_Active := FALSE;
    VerticalCutter_Active := FALSE;
    Running := FALSE;
    EmergencyStopped := FALSE; // Reset emergency stop flag after handling
END_IF;

// Regulate Feeder Units to maintain winding tension
IF Running THEN
    IF FeederUnit1_Speed < TensionThreshold1 THEN
        FeederUnit1_Speed := FeederUnit1_Speed + 1;
    ELSIF FeederUnit1_Speed > TensionThreshold1 THEN
        FeederUnit1_Speed := FeederUnit1_Speed - 1;
    END_IF;

    IF FeederUnit2_Speed < TensionThreshold2 THEN
        FeederUnit2_Speed := FeederUnit2_Speed + 1;
    ELSIF FeederUnit2_Speed > TensionThreshold2 THEN
        FeederUnit2_Speed := FeederUnit2_Speed - 1;
    END_IF;
END_IF;



