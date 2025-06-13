PROGRAM TrafficLightControl
VAR
    // Inputs
    Pedestrian_Button : BOOL; // Input from pedestrian crosswalk button
    Emergency_Vehicle_Detected : BOOL; // Input from emergency vehicle detector

    // Outputs
    Green_Light : BOOL; // Output to green light
    Yellow_Light : BOOL; // Output to yellow light
    Red_Light : BOOL; // Output to red light

    // State Machine Variables
    CurrentState : INT := 0;
    NORMAL_GREEN : CONST(INT) := 0;
    TO_YELLOW : CONST(INT) := 1;
    TO_RED : CONST(INT) := 2;
    PEDESTRIAN_CROSS : CONST(INT) := 3;
    EMERGENCY : CONST(INT) := 4;

    // Timers
    Timer_Green : TON; // Timer for green light duration
    Timer_Yellow : TON; // Timer for yellow light duration
    Timer_Red : TON; // Timer for red light duration

    // Edge Detection Triggers
    TRG_Pedestrian_Button : R_TRIG; // Rising edge trigger for pedestrian button
    TRG_Emergency_Vehicle_Detected : R_TRIG; // Rising edge trigger for emergency vehicle detection

    // Constants
    GREEN_DURATION : TIME := T#5s; // Duration of green light
    YELLOW_DURATION : TIME := T#2s; // Duration of yellow light
    RED_DURATION : TIME := T#5s; // Duration of red light
END_VAR

// Initialize triggers
TRG_Pedestrian_Button(CLK := Pedestrian_Button);
TRG_Emergency_Vehicle_Detected(CLK := Emergency_Vehicle_Detected);

// Main cyclic execution
CASE CurrentState OF
    NORMAL_GREEN:
        // Set lights
        Green_Light := TRUE;
        Yellow_Light := FALSE;
        Red_Light := FALSE;

        // Start green timer if not already running
        IF NOT Timer_Green.Q THEN
            Timer_Green(IN := TRUE);
            Timer_Green(PT := GREEN_DURATION);
        END_IF;

        // Transition to TO_YELLOW after green duration
        IF Timer_Green.Q THEN
            Timer_Green(IN := FALSE); // Reset timer
            CurrentState := TO_YELLOW;
        END_IF;

        // Handle pedestrian request
        IF TRG_Pedestrian_Button.Q THEN
            TRG_Pedestrian_Button(Q := FALSE); // Reset trigger
            Timer_Green(IN := FALSE); // Stop green timer early
            CurrentState := PEDESTRIAN_CROSS;
        END_IF;

        // Handle emergency vehicle detection
        IF TRG_Emergency_Vehicle_Detected.Q THEN
            TRG_Emergency_Vehicle_Detected(Q := FALSE); // Reset trigger
            Timer_Green(IN := FALSE); // Stop green timer early
            CurrentState := EMERGENCY;
        END_IF;

    TO_YELLOW:
        // Set lights
        Green_Light := FALSE;
        Yellow_Light := TRUE;
        Red_Light := FALSE;

        // Start yellow timer if not already running
        IF NOT Timer_Yellow.Q THEN
            Timer_Yellow(IN := TRUE);
            Timer_Yellow(PT := YELLOW_DURATION);
        END_IF;

        // Transition to TO_RED after yellow duration
        IF Timer_Yellow.Q THEN
            Timer_Yellow(IN := FALSE); // Reset timer
            CurrentState := TO_RED;
        END_IF;

    TO_RED:
        // Set lights
        Green_Light := FALSE;
        Yellow_Light := FALSE;
        Red_Light := TRUE;

        // Start red timer if not already running
        IF NOT Timer_Red.Q THEN
            Timer_Red(IN := TRUE);
            Timer_Red(PT := RED_DURATION);
        END_IF;

        // Transition to NORMAL_GREEN after red duration
        IF Timer_Red.Q THEN
            Timer_Red(IN := FALSE); // Reset timer
            CurrentState := NORMAL_GREEN;
        END_IF;

    PEDESTRIAN_CROSS:
        // Set lights
        Green_Light := FALSE;
        Yellow_Light := FALSE;
        Red_Light := TRUE;

        // Keep red light on while pedestrian crosses
        // Add additional logic here if needed (e.g., countdown timer)

        // After pedestrian crossing, transition back to TO_RED
        // For simplicity, assume immediate transition back to TO_RED
        CurrentState := TO_RED;

    EMERGENCY:
        // Set lights
        Green_Light := FALSE;
        Yellow_Light := FALSE;
        Red_Light := TRUE;

        // Keep red light on for emergency vehicles
        // Additional logic can be added here if needed (e.g., priority handling)

        // After emergency vehicle passes, transition back to NORMAL_GREEN
        // For simplicity, assume immediate transition back to NORMAL_GREEN
        IF NOT Emergency_Vehicle_Detected THEN
            CurrentState := NORMAL_GREEN;
        END_IF;

    ELSE
        // Default case (should not happen)
        CurrentState := NORMAL_GREEN;
END_CASE;
