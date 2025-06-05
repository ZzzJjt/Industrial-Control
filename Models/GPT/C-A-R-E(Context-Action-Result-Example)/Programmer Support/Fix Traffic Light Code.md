PROGRAM TrafficLightController
VAR
    // State definitions
    State : INT := 0; // 0=NORMAL, 1=TO_YELLOW, 2=TO_RED, 3=PEDESTRIAN_WAIT, 4=EMERGENCY

    // Timers
    tGreen    : TON;
    tYellow   : TON;
    tPed      : TON;

    // Timer duration constants
    GREEN_TIME     : TIME := T#10s;
    YELLOW_TIME    : TIME := T#3s;
    PEDESTRIAN_TIME: TIME := T#6s;

    // Inputs
    Pedestrian_Request : BOOL;
    Emergency_Signal   : BOOL;

    // Outputs
    Red    : BOOL := FALSE;
    Yellow : BOOL := FALSE;
    Green  : BOOL := FALSE;
    Pedestrian_Walk : BOOL := FALSE;
END_VAR

// Main state machine
CASE State OF

    // Normal Green Light Phase
    0: // NORMAL
        Red := FALSE;
        Yellow := FALSE;
        Green := TRUE;
        Pedestrian_Walk := FALSE;

        tGreen(IN := TRUE, PT := GREEN_TIME);
        IF tGreen.Q THEN
            tGreen(IN := FALSE);
            State := 1; // Transition to yellow
        ELSIF Pedestrian_Request THEN
            tGreen(IN := FALSE);
            State := 3; // Pedestrian override
        ELSIF Emergency_Signal THEN
            tGreen(IN := FALSE);
            State := 4; // Emergency override
        END_IF

    // Yellow Light Transition
    1: // TO_YELLOW
        Red := FALSE;
        Yellow := TRUE;
        Green := FALSE;

        tYellow(IN := TRUE, PT := YELLOW_TIME);
        IF tYellow.Q THEN
            tYellow(IN := FALSE);
            State := 2; // Go to red
        END_IF

    // Red Light Hold (before returning to green)
    2: // TO_RED
        Red := TRUE;
        Yellow := FALSE;
        Green := FALSE;

        // Small delay before transitioning back
        tGreen(IN := TRUE, PT := T#2s);
        IF tGreen.Q THEN
            tGreen(IN := FALSE);
            State := 0; // Back to normal cycle
        END_IF

    // Pedestrian Wait Phase
    3: // PEDESTRIAN_WAIT
        Red := TRUE;
        Yellow := FALSE;
        Green := FALSE;
        Pedestrian_Walk := TRUE;

        tPed(IN := TRUE, PT := PEDESTRIAN_TIME);
        IF tPed.Q THEN
            tPed(IN := FALSE);
            State := 0; // Resume normal
        END_IF

    // Emergency Override (keep green light for emergency vehicle)
    4: // EMERGENCY_OVERRIDE
        Red := FALSE;
        Yellow := FALSE;
        Green := TRUE;
        Pedestrian_Walk := FALSE;

        IF NOT Emergency_Signal THEN
            State := 1; // Resume normal via yellow
        END_IF

END_CASE
