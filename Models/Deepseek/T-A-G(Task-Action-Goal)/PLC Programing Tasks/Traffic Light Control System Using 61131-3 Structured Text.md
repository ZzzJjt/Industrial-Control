PROGRAM TrafficLightControl
VAR_INPUT
    PedestrianRequest : BOOL; // Input for pedestrian request button
    EmergencyDetected : BOOL; // Input for emergency vehicle detection
END_VAR

VAR_OUTPUT
    RedLight : BOOL := TRUE; // Output for red light
    GreenLight : BOOL := FALSE; // Output for green light
    YellowLight : BOOL := FALSE; // Output for yellow light
    PedestrianSignal : BOOL := FALSE; // Output for pedestrian crossing signal
END_VAR

VAR
    TrafficState : INT := 0; // 0=Red, 1=Green, 2=Yellow
    PedestrianActive : BOOL := FALSE; // Flag indicating active pedestrian request
    EmergencyActive : BOOL := FALSE; // Flag indicating active emergency condition
    StateTimer : TON; // Timer for state transitions
END_VAR

// Emergency override logic
IF EmergencyDetected THEN
    EmergencyActive := TRUE;
    TrafficState := 1; // Force green
    RedLight := FALSE;
    GreenLight := TRUE;
    YellowLight := FALSE;
    PedestrianSignal := FALSE;
    StateTimer(IN := FALSE); // Reset timer
ELSIF EmergencyActive THEN
    // Stay in green until emergency clears
    IF NOT EmergencyDetected THEN
        EmergencyActive := FALSE;
        TrafficState := 0; // Return to red
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        PedestrianSignal := FALSE;
        StateTimer(IN := FALSE); // Reset timer
    END_IF;
ELSE
    // Normal traffic light sequence
    CASE TrafficState OF
        0: // Red
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            PedestrianSignal := FALSE;
            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                IF PedestrianRequest THEN
                    PedestrianActive := TRUE;
                    StateTimer(IN := FALSE); // Extend red
                ELSE
                    TrafficState := 1; // Go to green
                    StateTimer(IN := FALSE);
                END_IF;
            END_IF;

        1: // Green
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;
            IF PedestrianActive THEN
                PedestrianSignal := TRUE;
            ELSE
                PedestrianSignal := FALSE;
            END_IF;
            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                TrafficState := 2; // Go to yellow
                StateTimer(IN := FALSE);
            END_IF;

        2: // Yellow
            RedLight := FALSE;
            GreenLight := FALSE;
            YellowLight := TRUE;
            PedestrianSignal := FALSE;
            StateTimer(IN := TRUE, PT := T#3s);
            IF StateTimer.Q THEN
                TrafficState := 0; // Return to red
                PedestrianRequest := FALSE;
                PedestrianActive := FALSE;
                StateTimer(IN := FALSE);
            END_IF;
    END_CASE;
END_IF;

// Additional comments for clarity
// - Implement standard signal sequencing using TrafficState variable.
// - Use TON timer to control the duration of each state: Red = 10s, Green = 10s, Yellow = 3s.
// - Handle pedestrian requests by extending the red time during the red phase.
// - Respond to emergency vehicle detection by forcing the system to switch to green immediately.
// - Control state transitions safely by resetting timers properly and handling edge cases with boolean flags.



