PROGRAM TrafficLightController
VAR
    // States: 0 = Red, 1 = Green, 2 = Yellow
    TrafficState       : INT := 0;

    // Inputs
    PedestrianRequest  : BOOL;     // Pedestrian push button
    EmergencyDetected  : BOOL;     // Emergency vehicle sensor

    // Internal flags
    PedestrianActive   : BOOL := FALSE;

    // Outputs
    RedLight           : BOOL := TRUE;
    GreenLight         : BOOL := FALSE;
    YellowLight        : BOOL := FALSE;

    // Timer
    StateTimer         : TON;
END_VAR

// === Emergency Vehicle Override ===
IF EmergencyDetected THEN
    TrafficState := 1;                    // Force Green
    PedestrianActive := FALSE;           // Cancel pedestrian logic
    StateTimer(IN := FALSE);             // Reset timer
ELSE

    CASE TrafficState OF

        // === RED STATE ===
        0:
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;

            StateTimer(IN := TRUE, PT := T#10s);

            IF StateTimer.Q THEN
                IF PedestrianRequest AND NOT PedestrianActive THEN
                    // Extend red to allow crossing
                    PedestrianActive := TRUE;
                    StateTimer(IN := FALSE); // Restart timer for crossing delay
                ELSE
                    TrafficState := 1;       // Move to GREEN
                    StateTimer(IN := FALSE);
                END_IF
            END_IF;

        // === GREEN STATE ===
        1:
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;

            StateTimer(IN := TRUE, PT := T#10s);

            IF StateTimer.Q THEN
                TrafficState := 2;           // Move to YELLOW
                StateTimer(IN := FALSE);
            END_IF;

        // === YELLOW STATE ===
        2:
            RedLight := FALSE;
            GreenLight := FALSE;
            YellowLight := TRUE;

            StateTimer(IN := TRUE, PT := T#3s);

            IF StateTimer.Q THEN
                TrafficState := 0;           // Return to RED
                PedestrianRequest := FALSE;  // Reset pedestrian logic
                PedestrianActive := FALSE;
                StateTimer(IN := FALSE);
            END_IF;

    END_CASE;
END_IF;
