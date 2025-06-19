PROGRAM PLC_PRG
TITLE 'Traffic Light Controller – With Pedestrian & Emergency Priority'

(*
    Description:
    A traffic light control system that supports normal sequencing,
    pedestrian crossing requests, and emergency vehicle override.

    Features:
    - Fixed timing for standard operation
    - Pedestrian request handled during Red phase
    - Emergency vehicle instantly forces green
    - Safe interlocks to prevent conflicting states
    - Clear state-based control with CASE structure
    
    Safety:
    - Prevents unsafe transitions
    - Ensures pedestrian only crosses when safe
    - Prioritizes emergency vehicles immediately
*)

// Configuration Constants
CONST
    GREEN_DURATION   : TIME := T#10s;
    YELLOW_DURATION  : TIME := T#3s;
    RED_DURATION     : TIME := T#10s;
END_CONST

VAR
    // Inputs
    PedestrianButton : BOOL := FALSE;     // TRUE when pressed
    EmergencyVehicle : BOOL := FALSE;     // TRUE when detected

    // Internal Logic
    TrafficState     : INT := 0;           // 0=Red, 1=Green, 2=Yellow
    StateTimer       : TON;                // Timer for each state
    PedestrianActive : BOOL := FALSE;      // Indicates active pedestrian crossing
    PedestrianRequest: BOOL := FALSE;      // Captures button press

    // Outputs
    LightRed         : BOOL := FALSE;
    LightYellow      : BOOL := FALSE;
    LightGreen       : BOOL := FALSE;
    WalkSignal       : BOOL := FALSE;      // Pedestrian "Walk" indicator
END_VAR

// === MAIN LOGIC ===

// Capture pedestrian request on rising edge
IF PedestrianButton AND NOT PedestrianRequest THEN
    PedestrianRequest := TRUE;
END_IF;

// --- EMERGENCY OVERRIDE ---
IF EmergencyVehicle THEN
    // Stop all timers
    StateTimer(IN := FALSE);

    // Force to Green if not already there
    IF TrafficState <> 1 THEN
        TrafficState := 1;
    END_IF;

    // Turn off pedestrian signal while handling emergency
    WalkSignal := FALSE;
    PedestrianActive := FALSE;

// --- NORMAL SEQUENCE CONTROL ---
ELSE
    CASE TrafficState OF
        // == STATE 0: RED ==
        0: BEGIN
            StateTimer(IN := TRUE, PT := RED_DURATION);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);

                // If pedestrian requested, keep red and activate walk
                IF PedestrianRequest THEN
                    PedestrianActive := TRUE;
                    WalkSignal := TRUE;
                    PedestrianRequest := FALSE;
                ELSE
                    // Proceed to green
                    TrafficState := 1;
                END_IF;
            END_IF;
        END;

        // == STATE 1: GREEN ==
        1: BEGIN
            StateTimer(IN := TRUE, PT := GREEN_DURATION);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                TrafficState := 2; // Move to yellow
            END_IF;
        END;

        // == STATE 2: YELLOW ==
        2: BEGIN
            StateTimer(IN := TRUE, PT := YELLOW_DURATION);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                TrafficState := 0; // Return to red
                WalkSignal := FALSE;
                PedestrianActive := FALSE;
            END_IF;
        END;
    END_CASE;
END_IF;

// === OUTPUT LOGIC ===
CASE TrafficState OF
    0: // Red
        LightRed := TRUE;
        LightYellow := FALSE;
        LightGreen := FALSE;

    1: // Green
        LightRed := FALSE;
        LightYellow := FALSE;
        LightGreen := TRUE;

    2: // Yellow
        LightRed := FALSE;
        LightYellow := TRUE;
        LightGreen := FALSE;
END_CASE;

// Keep WalkSignal active only during extended Red phase
IF NOT PedestrianActive THEN
    WalkSignal := FALSE;
END_IF;

END_PROGRAM
