VAR
    // Inputs
    Ped_Button           : BOOL;              // Pedestrian request button
    Emergency_Detect     : BOOL;              // Emergency vehicle detector

    // Traffic light outputs
    Light_Green          : BOOL := FALSE;
    Light_Yellow         : BOOL := FALSE;
    Light_Red            : BOOL := FALSE;
    Pedestrian_Walk      : BOOL := FALSE;

    // Internal state machine
    State                : INT := 0;          // 0=NORMAL_GREEN, 1=TO_YELLOW, 2=TO_RED, 3=PEDESTRIAN_CROSS, 4=EMERGENCY
    Next_State           : INT;
    
    // Edge detection
    Ped_Trig             : R_TRIG;
    Ped_Button_Rise      : BOOL;

    Emergency_Trig       : R_TRIG;
    Emergency_Rise       : BOOL;

    // Timers
    t_Yellow             : TON;
    t_Red                : TON;
    t_Pedestrian         : TON;
    t_Normal             : TON;

    // Timer trigger booleans
    Start_Yellow         : BOOL := FALSE;
    Start_Red            : BOOL := FALSE;
    Start_Pedestrian     : BOOL := FALSE;
    Start_Normal         : BOOL := FALSE;
END_VAR

// Rising edge detection
Ped_Trig(CLK := Ped_Button);
Ped_Button_Rise := Ped_Trig.Q;

Emergency_Trig(CLK := Emergency_Detect);
Emergency_Rise := Emergency_Trig.Q;

// Timer execution
t_Yellow(IN := Start_Yellow, PT := T#5s);
t_Red(IN := Start_Red, PT := T#2s);
t_Pedestrian(IN := Start_Pedestrian, PT := T#6s);
t_Normal(IN := Start_Normal, PT := T#10s);

// State machine
CASE State OF

    // NORMAL GREEN phase
    0:
        Light_Green := TRUE;
        Light_Yellow := FALSE;
        Light_Red := FALSE;
        Pedestrian_Walk := FALSE;

        Start_Normal := TRUE;
        IF Emergency_Rise THEN
            Start_Normal := FALSE;
            State := 4; // Emergency override
        ELSIF Ped_Button_Rise THEN
            Start_Normal := FALSE;
            State := 1; // Go to yellow transition
        ELSIF t_Normal.Q THEN
            Start_Normal := FALSE;
            State := 1;
        END_IF

    // Transition to YELLOW
    1:
        Light_Green := FALSE;
        Light_Yellow := TRUE;
        Light_Red := FALSE;
        Pedestrian_Walk := FALSE;

        Start_Yellow := TRUE;
        IF t_Yellow.Q THEN
            Start_Yellow := FALSE;
            State := 2;
        END_IF

    // Transition to RED
    2:
        Light_Green := FALSE;
        Light_Yellow := FALSE;
        Light_Red := TRUE;
        Pedestrian_Walk := FALSE;

        Start_Red := TRUE;
        IF t_Red.Q THEN
            Start_Red := FALSE;
            State := 3;
        END_IF

    // Pedestrian crossing phase
    3:
        Light_Green := FALSE;
        Light_Yellow := FALSE;
        Light_Red := TRUE;
        Pedestrian_Walk := TRUE;

        Start_Pedestrian := TRUE;
        IF t_Pedestrian.Q THEN
            Start_Pedestrian := FALSE;
            State := 0;
        END_IF

    // Emergency vehicle override: GREEN immediately
    4:
        Light_Green := TRUE;
        Light_Yellow := FALSE;
        Light_Red := FALSE;
        Pedestrian_Walk := FALSE;

        // Stay green as long as emergency is active
        IF NOT Emergency_Detect THEN
            State := 0; // Return to normal
        END_IF

ELSE
    // Fallback to safe state
    Light_Green := FALSE;
    Light_Yellow := FALSE;
    Light_Red := TRUE;
    Pedestrian_Walk := FALSE;
    State := 0;
END_CASE
