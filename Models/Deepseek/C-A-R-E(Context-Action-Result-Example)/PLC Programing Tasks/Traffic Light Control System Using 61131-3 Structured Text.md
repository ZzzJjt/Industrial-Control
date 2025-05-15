PROGRAM TrafficLightControl
VAR_INPUT
    PedestrianRequest : BOOL; // Pedestrian push button input
    EmergencyDetected : BOOL; // Emergency vehicle sensor input
END_VAR

VAR_OUTPUT
    RedLight : BOOL := TRUE;   // Red light output
    GreenLight : BOOL := FALSE; // Green light output
    YellowLight : BOOL := FALSE; // Yellow light output
    PedestrianCrossing : BOOL := FALSE; // Pedestrian crossing active
END_VAR

VAR
    TrafficState : INT := 0; // 0 = Red, 1 = Green, 2 = Yellow
    PedestrianActive : BOOL := FALSE; // Flag indicating pedestrian crossing is active
    StateTimer : TON; // Timer for state transitions
    LastEmergencyDetected : BOOL; // Previous state of EmergencyDetected
    LastPedestrianRequest : BOOL; // Previous state of PedestrianRequest
END_VAR

// Handle emergency vehicle override
IF EmergencyDetected AND NOT LastEmergencyDetected THEN
    TrafficState := 1; // Force green
    PedestrianActive := FALSE; // Reset pedestrian crossing
    StateTimer(IN := FALSE); // Reset timer
ELSE
    CASE TrafficState OF
        0: // Red
            IF PedestrianRequest AND NOT LastPedestrianRequest THEN
                PedestrianActive := TRUE; // Extend red for crossing
                StateTimer(IN := TRUE, PT := T#15s); // Longer red time for pedestrians
            ELSE
                StateTimer(IN := TRUE, PT := T#10s); // Normal red time
            END_IF;
            
            IF StateTimer.Q THEN
                IF PedestrianActive THEN
                    StateTimer(IN := FALSE); // Continue holding red
                ELSE
                    TrafficState := 1; // Go to green
                    StateTimer(IN := FALSE);
                END_IF;
            END_IF;

        1: // Green
            StateTimer(IN := TRUE, PT := T#10s); // Green duration
            
            IF StateTimer.Q THEN
                TrafficState := 2; // Go to yellow
                StateTimer(IN := FALSE);
            END_IF;

        2: // Yellow
            StateTimer(IN := TRUE, PT := T#3s); // Yellow duration
            
            IF StateTimer.Q THEN
                TrafficState := 0; // Return to red
                PedestrianRequest := FALSE;
                PedestrianActive := FALSE;
                StateTimer(IN := FALSE);
            END_IF;
    END_CASE;
END_IF;

// Update previous states
LastEmergencyDetected := EmergencyDetected;
LastPedestrianRequest := PedestrianRequest;

// Set light outputs based on current state
CASE TrafficState OF
    0: // Red
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
    1: // Green
        RedLight := FALSE;
        GreenLight := TRUE;
        YellowLight := FALSE;
    2: // Yellow
        RedLight := FALSE;
        GreenLight := FALSE;
        YellowLight := TRUE;
END_CASE;

// Set pedestrian crossing indicator
PedestrianCrossing := PedestrianActive;

// Inline comments explaining the logic:
// The program manages a traffic light system with standard sequences: Red → Green → Yellow.
// It handles pedestrian push buttons by inserting a safe red light hold for crossing.
// It detects emergency vehicles and immediately grants a green signal to clear the path.
// It resumes normal traffic cycles after special events and ensures non-conflicting transitions with timer-based state control.



