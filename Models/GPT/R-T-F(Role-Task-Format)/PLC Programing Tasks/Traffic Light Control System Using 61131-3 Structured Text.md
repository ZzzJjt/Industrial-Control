PROGRAM TrafficLightControl
VAR
    // Inputs
    PedestrianRequest : BOOL;            // Pedestrian button pressed
    EmergencyDetected : BOOL;            // Emergency vehicle detected

    // Outputs
    RedLight     : BOOL := FALSE;
    YellowLight  : BOOL := FALSE;
    GreenLight   : BOOL := FALSE;

    // Internal State
    TrafficState : INT := 0;             // 0 = Red, 1 = Green, 2 = Yellow
    PedestrianActive : BOOL := FALSE;

    // Timers
    StateTimer : TON;
END_VAR

// --- Emergency Vehicle Override ---
// If detected, force green and reset all timing
IF EmergencyDetected THEN
    RedLight := FALSE;
    YellowLight := FALSE;
    GreenLight := TRUE;
    TrafficState := 1; // Force state to Green
    StateTimer(IN := FALSE); // Cancel any ongoing timer
ELSE
    CASE TrafficState OF
        0: // Red
            RedLight := TRUE;
            YellowLight := FALSE;
            GreenLight := FALSE;

            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                IF PedestrianRequest THEN
                    PedestrianActive := TRUE;
                    StateTimer(IN := FALSE); // Extend red
                ELSIF PedestrianActive THEN
                    // Maintain red for extended duration
                    StateTimer(IN := TRUE, PT := T#5s);
                    IF StateTimer.Q THEN
                        PedestrianActive := FALSE;
                        TrafficState := 1; // Proceed to green
                        StateTimer(IN := FALSE);
                    END_IF;
                ELSE
                    TrafficState := 1; // Go to green
                    StateTimer(IN := FALSE);
                END_IF;
            END_IF;

        1: // Green
            RedLight := FALSE;
            YellowLight := FALSE;
            GreenLight := TRUE;

            StateTimer(IN := TRUE, PT := T#15s);
            IF StateTimer.Q THEN
                TrafficState := 2; // Go to yellow
                StateTimer(IN := FALSE);
            END_IF;

        2: // Yellow
            RedLight := FALSE;
            YellowLight := TRUE;
            GreenLight := FALSE;

            StateTimer(IN := TRUE, PT := T#3s);
            IF StateTimer.Q THEN
                TrafficState := 0; // Return to red
                StateTimer(IN := FALSE);
            END_IF;
    END_CASE;
END_IF;
