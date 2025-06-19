FUNCTION_BLOCK FB_TrafficLightControl
VAR_INPUT
    // Inputs
    PedestrianRequest : BOOL;   // TRUE when pedestrian presses the button
    EmergencyDetected : BOOL;   // TRUE when emergency vehicle is detected
END_VAR

VAR_OUTPUT
    // Output Light State
    TrafficState : INT := 0;    // 0 = Red, 1 = Green, 2 = Yellow

    // Diagnostic Flags
    PedestrianActive : BOOL := FALSE;
    EmergencyActive  : BOOL := FALSE;
END_VAR

VAR
    // Internal Logic
    StateTimer : TON;           // Timer used to manage state durations
    LastEmergency : BOOL := FALSE;
END_VAR


// --- STEP 1: Emergency Vehicle Override ---
IF EmergencyDetected THEN
    // Activate emergency mode
    EmergencyActive := TRUE;

    // Clear all other flags
    PedestrianRequest := FALSE;
    PedestrianActive := FALSE;

    // Force green light immediately
    TrafficState := 1;
    StateTimer(IN := FALSE); // Stop any running timer

ELSIF LastEmergency AND NOT EmergencyDetected THEN
    // Emergency has cleared — reset everything and restart from Red
    EmergencyActive := FALSE;
    TrafficState := 0;
    StateTimer(IN := FALSE);
    StateTimer(ET := T#0s);

END_IF;

// --- STEP 2: Standard Operation Mode (No Emergency) ---
CASE TrafficState OF
    0: // RED LIGHT STATE
        StateTimer(IN := TRUE, PT := T#10s);

        IF StateTimer.Q THEN
            // If pedestrian requested and not in emergency, extend red
            IF PedestrianRequest AND NOT EmergencyActive THEN
                PedestrianActive := TRUE;
                StateTimer(IN := FALSE);
                StateTimer(ET := T#0s);
            ELSE
                // No pedestrian — move to Green
                StateTimer(IN := FALSE);
                StateTimer(ET := T#0s);
                TrafficState := 1;
            END_IF;
        END_IF;

    1: // GREEN LIGHT STATE
        StateTimer(IN := TRUE, PT := T#10s);

        IF StateTimer.Q THEN
            // Move to Yellow after timeout
            StateTimer(IN := FALSE);
            StateTimer(ET := T#0s);
            TrafficState := 2;
        END_IF;

    2: // YELLOW LIGHT STATE
        StateTimer(IN := TRUE, PT := T#3s);

        IF StateTimer.Q THEN
            // Return to Red and clear pedestrian request
            StateTimer(IN := FALSE);
            StateTimer(ET := T#0s);
            PedestrianRequest := FALSE;
            PedestrianActive := FALSE;
            TrafficState := 0;
        END_IF;
END_CASE;

// Store previous emergency state for edge detection
LastEmergency := EmergencyDetected;

PROGRAM PLC_PRG
VAR
    TrafficCtrl : FB_TrafficLightControl;

    // Input Simulation
    CrossButton : BOOL := FALSE;
    AmbulanceDetected : BOOL := FALSE;

    // Outputs / Status
    LightState : INT;
    IsPedestrianCrossing : BOOL;
    IsEmergencyMode : BOOL;
END_VAR

// Call the function block
TrafficCtrl(
    PedestrianRequest := CrossButton,
    EmergencyDetected := AmbulanceDetected,

    TrafficState => LightState,
    PedestrianActive => IsPedestrianCrossing,
    EmergencyActive => IsEmergencyMode
);
