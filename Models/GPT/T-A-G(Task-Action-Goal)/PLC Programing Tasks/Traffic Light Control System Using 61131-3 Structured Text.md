VAR
    // Traffic State: 0 = Red, 1 = Green, 2 = Yellow
    TrafficState : INT := 0;

    // Inputs
    PedestrianRequest    : BOOL;      // Button press for pedestrian crossing
    EmergencyDetected    : BOOL;      // Emergency vehicle detection

    // Internal Flags
    PedestrianActive     : BOOL := FALSE;  // Pedestrian logic in progress
    TimerRunning         : BOOL := FALSE;

    // Timer
    StateTimer           : TON;

    // Outputs
    RedLight             : BOOL := FALSE;
    GreenLight           : BOOL := FALSE;
    YellowLight          : BOOL := FALSE;
END_VAR

//-------------------------//
// Emergency Handling Logic
//-------------------------//
IF EmergencyDetected THEN
    TrafficState := 1; // Force green
    GreenLight := TRUE;
    RedLight := FALSE;
    YellowLight := FALSE;
    StateTimer(IN := FALSE);
    TimerRunning := FALSE;
ELSE
    //----------------------//
    // Traffic State Machine
    //----------------------//
    CASE TrafficState OF

        //------ RED ------//
        0:
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;

            IF NOT TimerRunning THEN
                StateTimer(IN := TRUE, PT := T#10s);
                TimerRunning := TRUE;
            END_IF;

            IF StateTimer.Q THEN
                IF PedestrianRequest THEN
                    PedestrianActive := TRUE;
                    // Extend red phase, reset timer
                    StateTimer(IN := FALSE);
                    TimerRunning := FALSE;
                ELSE
                    // Transition to GREEN
                    TrafficState := 1;
                    StateTimer(IN := FALSE);
                    TimerRunning := FALSE;
                END_IF;
            END_IF;

        //------ GREEN ------//
        1:
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;

            IF NOT TimerRunning THEN
                StateTimer(IN := TRUE, PT := T#10s);
                TimerRunning := TRUE;
            END_IF;

            IF StateTimer.Q THEN
                TrafficState := 2; // Go to YELLOW
                StateTimer(IN := FALSE);
                TimerRunning := FALSE;
            END_IF;

        //------ YELLOW ------//
        2:
            RedLight := FALSE;
            GreenLight := FALSE;
            YellowLight := TRUE;

            IF NOT TimerRunning THEN
                StateTimer(IN := TRUE, PT := T#3s);
                TimerRunning := TRUE;
            END_IF;

            IF StateTimer.Q THEN
                TrafficState := 0; // Back to RED
                PedestrianActive := FALSE;
                PedestrianRequest := FALSE;
                StateTimer(IN := FALSE);
                TimerRunning := FALSE;
            END_IF;

    END_CASE
END_IF;
