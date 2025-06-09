FUNCTION_BLOCK TrafficLightController
VAR_INPUT
    PedestrianRequest    : BOOL; // Pedestrian crossing button
    EmergencyDetected    : BOOL; // Emergency vehicle presence
END_VAR

VAR_OUTPUT
    TrafficState         : INT := 0; // 0 = Red, 1 = Green, 2 = Yellow
    PedestrianActive     : BOOL := FALSE;
END_VAR

VAR
    StateTimer           : TON;
    RedHold              : BOOL := FALSE; // Indicates extended red for pedestrians
END_VAR

// --- Emergency override logic ---
IF EmergencyDetected THEN
    TrafficState := 1; // Force green
    StateTimer(IN := FALSE); // Cancel any running timer
    PedestrianActive := FALSE;
    RedHold := FALSE;

ELSE

    CASE TrafficState OF
        // --- Red ---
        0:
            IF NOT StateTimer.IN THEN
                // Normal red duration or extended if pedestrian requested
                IF PedestrianRequest THEN
                    RedHold := TRUE;
                    StateTimer(IN := TRUE, PT := T#13s); // Extend red (10s + 3s)
                    PedestrianActive := TRUE;
                ELSE
                    StateTimer(IN := TRUE, PT := T#10s);
                END_IF
            END_IF

            IF StateTimer.Q THEN
                TrafficState := 1; // Move to Green
                StateTimer(IN := FALSE);
                PedestrianRequest := FALSE;
                RedHold := FALSE;
            END_IF

        // --- Green ---
        1:
            IF NOT StateTimer.IN THEN
                StateTimer(IN := TRUE, PT := T#10s);
            END_IF

            IF StateTimer.Q THEN
                TrafficState := 2; // Move to Yellow
                StateTimer(IN := FALSE);
            END_IF

        // --- Yellow ---
        2:
            IF NOT StateTimer.IN THEN
                StateTimer(IN := TRUE, PT := T#3s);
            END_IF

            IF StateTimer.Q THEN
                TrafficState := 0; // Return to Red
                StateTimer(IN := FALSE);
                PedestrianActive := FALSE;
            END_IF
    END_CASE
END_IF
