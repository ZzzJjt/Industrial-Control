PROGRAM TrafficLightControl
VAR_INPUT
    PedestrianButton : BOOL; // Input from pedestrian button
    EmergencyVehicle : BOOL; // Input indicating emergency vehicle presence
END_VAR

VAR_OUTPUT
    GreenLight : BOOL;
    YellowLight : BOOL;
    RedLight : BOOL;
END_VAR

VAR
    State : INT := 0; // Finite state machine state variable
    TimerGreen : TON; // Timer for green phase
    TimerYellow : TON; // Timer for yellow phase
    TimerRed : TON; // Timer for red phase
    TimerPedWait : TON; // Timer for pedestrian wait phase
    RTrigPedestrian : R_TRIG; // Rising edge detector for pedestrian button
    RTrigEmergency : R_TRIG; // Rising edge detector for emergency vehicle
    PedRequested : BOOL; // Flag to indicate pedestrian request
    EmergencyOverride : BOOL; // Flag to indicate emergency override
END_VAR

CONST
    GREEN_PHASE : INT := 0;
    YELLOW_PHASE : INT := 1;
    RED_PHASE : INT := 2;
    PEDESTRIAN_WAIT : INT := 3;
    EMERGENCY_OVERRIDE : INT := 4;

// Initialize timers
TimerGreen(IN := FALSE);
TimerYellow(IN := FALSE);
TimerRed(IN := FALSE);
TimerPedWait(IN := FALSE);

// Detect rising edges for pedestrian button and emergency vehicle
RTrigPedestrian(CLK := PedestrianButton);
RTrigEmergency(CLK := EmergencyVehicle);

// Main FSM logic
CASE State OF
    GREEN_PHASE:
        // Start green timer if not already started
        IF NOT TimerGreen.Q THEN
            TimerGreen(IN := TRUE);
            TimerGreen(PresetTime := T#5s); // Example duration for green phase
        END_IF;

        // Check for timeout or emergency override
        IF TimerGreen.Q OR EmergencyOverride THEN
            TimerGreen(IN := FALSE);
            State := YELLOW_PHASE;
        ELSIF PedRequested THEN
            TimerGreen(IN := FALSE);
            State := PEDESTRIAN_WAIT;
        END_IF;

    YELLOW_PHASE:
        // Start yellow timer if not already started
        IF NOT TimerYellow.Q THEN
            TimerYellow(IN := TRUE);
            TimerYellow(PresetTime := T#2s); // Example duration for yellow phase
        END_IF;

        // Check for timeout
        IF TimerYellow.Q THEN
            TimerYellow(IN := FALSE);
            State := RED_PHASE;
        END_IF;

    RED_PHASE:
        // Start red timer if not already started
        IF NOT TimerRed.Q THEN
            TimerRed(IN := TRUE);
            TimerRed(PresetTime := T#5s); // Example duration for red phase
        END_IF;

        // Check for timeout or emergency override
        IF TimerRed.Q OR EmergencyOverride THEN
            TimerRed(IN := FALSE);
            State := GREEN_PHASE;
        ELSIF PedRequested THEN
            TimerRed(IN := FALSE);
            State := PEDESTRIAN_WAIT;
        END_IF;

    PEDESTRIAN_WAIT:
        // Start pedestrian wait timer if not already started
        IF NOT TimerPedWait.Q THEN
            TimerPedWait(IN := TRUE);
            TimerPedWait(PresetTime := T#10s); // Example duration for pedestrian wait phase
        END_IF;

        // Check for timeout
        IF TimerPedWait.Q THEN
            TimerPedWait(IN := FALSE);
            State := RED_PHASE;
            PedRequested := FALSE;
        END_IF;

    EMERGENCY_OVERRIDE:
        // Set lights immediately to green
        GreenLight := TRUE;
        YellowLight := FALSE;
        RedLight := FALSE;

        // Wait until emergency condition clears
        IF NOT EmergencyOverride THEN
            State := GREEN_PHASE;
        END_IF;

    ELSE
        State := GREEN_PHASE; // Default state
END_CASE;

// Handle rising edge detections
IF RTrigPedestrian.Q THEN
    PedRequested := TRUE;
END_IF;

IF RTrigEmergency.Q THEN
    EmergencyOverride := TRUE;
END_IF;

// Update light outputs based on current state
CASE State OF
    GREEN_PHASE:
        GreenLight := TRUE;
        YellowLight := FALSE;
        RedLight := FALSE;

    YELLOW_PHASE:
        GreenLight := FALSE;
        YellowLight := TRUE;
        RedLight := FALSE;

    RED_PHASE:
        GreenLight := FALSE;
        YellowLight := FALSE;
        RedLight := TRUE;

    PEDESTRIAN_WAIT:
        GreenLight := FALSE;
        YellowLight := FALSE;
        RedLight := TRUE;

    EMERGENCY_OVERRIDE:
        GreenLight := TRUE;
        YellowLight := FALSE;
        RedLight := FALSE;

    ELSE
        GreenLight := FALSE;
        YellowLight := FALSE;
        RedLight := TRUE; // Default to red in unknown states
END_CASE;

// Reset emergency override when cleared
IF NOT EmergencyVehicle THEN
    EmergencyOverride := FALSE;
END_IF;



