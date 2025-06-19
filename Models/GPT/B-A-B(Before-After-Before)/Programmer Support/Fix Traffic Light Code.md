VAR
    State                : INT := 0; // 0: Green, 1: Yellow, 2: Red, 3: Pedestrian, 4: Emergency
    Timer_Transition     : TON;
    Timer_Duration       : TIME;

    pedestrianButton     : BOOL;  // Input
    emergencyDetected    : BOOL;  // Input

    Light_Red            : BOOL;
    Light_Yellow         : BOOL;
    Light_Green          : BOOL;
    Pedestrian_Signal    : BOOL;
END_VAR

// Timer instance logic
Timer_Transition(IN := FALSE, PT := Timer_Duration);

// Main state machine
CASE State OF

    0: // Green
        Light_Green := TRUE;
        Light_Yellow := FALSE;
        Light_Red := FALSE;
        Pedestrian_Signal := FALSE;

        IF emergencyDetected THEN
            State := 4; // Switch to Emergency
        ELSIF pedestrianButton THEN
            Timer_Duration := T#3s;
            Timer_Transition.IN := TRUE;
            State := 1; // Go to Yellow
        END_IF

    1: // Yellow (transition)
        Light_Green := FALSE;
        Light_Yellow := TRUE;
        Light_Red := FALSE;

        Timer_Transition(IN := TRUE);
        IF Timer_Transition.Q THEN
            Timer_Transition.IN := FALSE;
            Timer_Transition.Q := FALSE;
            State := 2; // Go to Red
        END_IF

    2: // Red before pedestrian cross
        Light_Green := FALSE;
        Light_Yellow := FALSE;
        Light_Red := TRUE;

        Timer_Duration := T#2s;
        Timer_Transition(IN := TRUE);
        IF Timer_Transition.Q THEN
            Timer_Transition.IN := FALSE;
            Timer_Transition.Q := FALSE;
            State := 3; // Pedestrian cross
        END_IF

    3: // Pedestrian Cross
        Light_Green := FALSE;
        Light_Yellow := FALSE;
        Light_Red := TRUE;
        Pedestrian_Signal := TRUE;

        Timer_Duration := T#5s;
        Timer_Transition(IN := TRUE);
        IF Timer_Transition.Q THEN
            Timer_Transition.IN := FALSE;
            Timer_Transition.Q := FALSE;
            Pedestrian_Signal := FALSE;
            State := 0; // Back to green
        END_IF

    4: // Emergency Mode – All red or blinking
        Light_Green := FALSE;
        Light_Yellow := FALSE;
        Light_Red := TRUE;
        Pedestrian_Signal := FALSE;

        IF NOT emergencyDetected THEN
            State := 0; // Resume normal cycle
        END_IF

    ELSE
        // Fail-safe
        Light_Green := FALSE;
        Light_Yellow := FALSE;
        Light_Red := TRUE;
        Pedestrian_Signal := FALSE;
        State := 0;
END_CASE
