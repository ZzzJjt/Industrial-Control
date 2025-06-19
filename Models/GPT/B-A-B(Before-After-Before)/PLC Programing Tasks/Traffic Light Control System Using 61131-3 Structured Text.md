VAR
    // Input signals
    PedestrianRequest : BOOL;          // Button press for crosswalk
    EmergencyDetected : BOOL;          // Emergency vehicle detected

    // Timer instance
    StateTimer : TON;

    // State management
    TrafficState : INT := 0;           // 0=Red, 1=Green, 2=Yellow
    PedestrianActive : BOOL := FALSE;

    // Outputs (example)
    RedLight : BOOL := FALSE;
    GreenLight : BOOL := FALSE;
    YellowLight : BOOL := FALSE;
    PedestrianSignal : BOOL := FALSE;
END_VAR

// === Emergency Priority Override ===
IF EmergencyDetected THEN
    TrafficState := 1;                     // Force green light
    RedLight := FALSE;
    YellowLight := FALSE;
    GreenLight := TRUE;
    PedestrianSignal := FALSE;
    StateTimer(IN := FALSE);              // Cancel any active timer
ELSE
    CASE TrafficState OF

        // === RED STATE ===
        0:
            RedLight := TRUE;
            YellowLight := FALSE;
            GreenLight := FALSE;
            StateTimer(IN := TRUE, PT := T#10s);
            
            // Handle pedestrian request during red
            IF PedestrianRequest THEN
                PedestrianSignal := TRUE;
                PedestrianActive := TRUE;
            END_IF
            
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                IF PedestrianActive THEN
                    // Extend red for pedestrian, reset flag
                    PedestrianSignal := FALSE;
                    PedestrianActive := FALSE;
                ELSE
                    TrafficState := 1; // Transition to green
                END_IF
            END_IF;

        // === GREEN STATE ===
        1:
            RedLight := FALSE;
            YellowLight := FALSE;
            GreenLight := TRUE;
            PedestrianSignal := FALSE;
            StateTimer(IN := TRUE, PT := T#10s);

            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                TrafficState := 2; // Transition to yellow
            END_IF;

        // === YELLOW STATE ===
        2:
            RedLight := FALSE;
            YellowLight := TRUE;
            GreenLight := FALSE;
            PedestrianSignal := FALSE;
            StateTimer(IN := TRUE, PT := T#3s);

            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                TrafficState := 0; // Return to red
                PedestrianRequest := FALSE;
            END_IF;

    END_CASE;
END_IF;
