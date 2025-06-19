PROGRAM TrafficLightControl
VAR
    TrafficState : INT := 0; // 0 = Red, 1 = Green, 2 = Yellow
    PedestrianRequest : BOOL; // Input from pedestrian push button
    EmergencyDetected : BOOL; // Input from emergency vehicle sensor
    PedestrianActive : BOOL := FALSE; // Flag indicating pedestrian crossing active
    StateTimer : TON; // Timer for state transitions
END_VAR

// Emergency mode: override with green
IF EmergencyDetected THEN
    TrafficState := 1; // Force green
    StateTimer(IN := FALSE); // Reset timer
    PedestrianActive := FALSE; // Deactivate pedestrian crossing
ELSE
    CASE TrafficState OF
        0: // Red
            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                IF PedestrianRequest THEN
                    PedestrianActive := TRUE; // Extend red for pedestrians
                    StateTimer(IN := FALSE); // Reset timer
                ELSE
                    TrafficState := 1; // Switch to green
                    StateTimer(IN := FALSE); // Reset timer
                END_IF;
            END_IF;

        1: // Green
            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                TrafficState := 2; // Switch to yellow
                StateTimer(IN := FALSE); // Reset timer
            END_IF;

        2: // Yellow
            StateTimer(IN := TRUE, PT := T#3s);
            IF StateTimer.Q THEN
                TrafficState := 0; // Switch back to red
                PedestrianRequest := FALSE; // Clear pedestrian request
                PedestrianActive := FALSE; // Deactivate pedestrian crossing
                StateTimer(IN := FALSE); // Reset timer
            END_IF;
    END_CASE;
END_IF;

// Additional logic for pedestrian crossing
IF PedestrianActive THEN
    StateTimer(IN := TRUE, PT := T#15s); // Extended red time for pedestrians
    IF StateTimer.Q THEN
        PedestrianActive := FALSE; // End pedestrian crossing
        StateTimer(IN := FALSE); // Reset timer
    END_IF;
END_IF;
