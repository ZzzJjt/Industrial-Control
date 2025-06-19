VAR
    // Inputs
    Pedestrian_Button : BOOL;
    Emergency_Input   : BOOL;

    // Outputs
    Red_Light    : BOOL;
    Yellow_Light : BOOL;
    Green_Light  : BOOL;

    // State control
    State : INT := 0; // 0: GREEN, 1: YELLOW, 2: RED, 3: PEDESTRIAN_WAIT, 4: EMERGENCY

    // Timer instance and flags
    Phase_Timer : TON;
    Timer_Duration : TIME;

    // Rising edge detection
    Pedestrian_Trig : R_TRIG;
    Emergency_Trig  : R_TRIG;
END_VAR

// Rising edge detection
Pedestrian_Trig(CLK := Pedestrian_Button);
Emergency_Trig(CLK := Emergency_Input);

// Emergency overrides everything
IF Emergency_Trig.Q THEN
    State := 4; // EMERGENCY
END_IF

// FSM State Machine
CASE State OF

    0: // GREEN PHASE
        Red_Light := FALSE;
        Yellow_Light := FALSE;
        Green_Light := TRUE;

        Timer_Duration := T#10s;
        Phase_Timer(IN := TRUE, PT := Timer_Duration);

        IF Pedestrian_Trig.Q THEN
            State := 1; // Go to yellow
            Phase_Timer(IN := FALSE); // Reset timer
        ELSIF Phase_Timer.Q THEN
            State := 1; // Time's up, switch to yellow
            Phase_Timer(IN := FALSE);
        END_IF

    1: // YELLOW PHASE
        Red_Light := FALSE;
        Yellow_Light := TRUE;
        Green_Light := FALSE;

        Timer_Duration := T#3s;
        Phase_Timer(IN := TRUE, PT := Timer_Duration);

        IF Phase_Timer.Q THEN
            State := 2; // RED
            Phase_Timer(IN := FALSE);
        END_IF

    2: // RED PHASE
        Red_Light := TRUE;
        Yellow_Light := FALSE;
        Green_Light := FALSE;

        Timer_Duration := T#5s;
        Phase_Timer(IN := TRUE, PT := Timer_Duration);

        IF Phase_Timer.Q THEN
            State := 0; // Back to green
            Phase_Timer(IN := FALSE);
        END_IF

    3: // PEDESTRIAN_WAIT (can be folded into RED state if desired)
        // For extension: insert wait for pedestrian clearance
        // For now, handled same as RED

    4: // EMERGENCY OVERRIDE
        Red_Light := FALSE;
        Yellow_Light := FALSE;
        Green_Light := TRUE;

        // Exit emergency only when Emergency_Input is off
        IF NOT Emergency_Input THEN
            State := 0; // Resume normal cycle
        END_IF

ELSE
    State := 0; // Default fallback
END_CASE
