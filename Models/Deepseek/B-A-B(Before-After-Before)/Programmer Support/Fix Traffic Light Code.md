TYPE E_TrafficState:
    (
    NORMAL,
    TO_YELLOW,
    TO_RED,
    PEDESTRIAN,
    EMERGENCY
    ) INT;
END_TYPE


PROGRAM PLC_PRG
VAR
    // Inputs
    pedestrianButtonPressed : BOOL := FALSE; // From input signal
    emergencyVehicleDetected : BOOL := FALSE; // From sensor/detection system

    // Timers
    trafficTimer      : TON; // Timer for phases
    pedWaitTimer      : TON; // Timer for pedestrian waiting
    emergencyHoldTime : TON; // Hold in emergency mode

    // State tracking
    currentState : E_TrafficState := E_TrafficState.NORMAL;
    nextState    : E_TrafficState := E_TrafficState.NORMAL;

    // Edge detection
    btnPedEdge : R_TRIG;
    btnEmergEdge : R_TRIG;

    // Outputs
    lightNS_Red    : BOOL := FALSE;
    lightNS_Yellow : BOOL := FALSE;
    lightNS_Green  : BOOL := FALSE;
    lightEW_Red    : BOOL := FALSE;
    lightEW_Yellow : BOOL := FALSE;
    lightEW_Green  : BOOL := FALSE;
    crosswalkLight : BOOL := FALSE;
END_VAR

btnPedEdge(CLK := pedestrianButtonPressed);
btnEmergEdge(CLK := emergencyVehicleDetected);

CASE currentState OF

    // --- STATE: NORMAL ---
    E_TrafficState.NORMAL:
        lightNS_Red := FALSE;
        lightNS_Yellow := FALSE;
        lightNS_Green := TRUE;
        lightEW_Red := TRUE;
        lightEW_Yellow := FALSE;
        lightEW_Green := FALSE;
        crosswalkLight := FALSE;

        IF btnPedEdge.Q THEN
            nextState := E_TrafficState.TO_YELLOW;
            trafficTimer(IN := TRUE, PT := T#5s); // NS Green -> Yellow
        ELSIF btnEmergEdge.Q THEN
            nextState := E_TrafficState.EMERGENCY;
        END_IF;

    // --- STATE: TO_YELLOW (Green -> Red) ---
    E_TrafficState.TO_YELLOW:
        lightNS_Red := FALSE;
        lightNS_Yellow := TRUE;
        lightNS_Green := FALSE;
        lightEW_Red := TRUE;
        lightEW_Yellow := FALSE;
        lightEW_Green := FALSE;
        crosswalkLight := FALSE;

        trafficTimer(IN := TRUE, PT := T#2s);

        IF trafficTimer.Q THEN
            trafficTimer(IN := FALSE);
            nextState := E_TrafficState.TO_RED;
        END_IF;

    // --- STATE: TO_RED (Yellow -> Red, then wait for pedestrian) ---
    E_TrafficState.TO_RED:
        lightNS_Red := TRUE;
        lightNS_Yellow := FALSE;
        lightNS_Green := FALSE;
        lightEW_Red := TRUE;
        lightEW_Yellow := FALSE;
        lightEW_Green := FALSE;
        crosswalkLight := FALSE;

        trafficTimer(IN := TRUE, PT := T#3s);

        IF trafficTimer.Q THEN
            trafficTimer(IN := FALSE);
            nextState := E_TrafficState.PEDESTRIAN;
        END_IF;

    // --- STATE: PEDESTRIAN (Allow safe crossing) ---
    E_TrafficState.PEDESTRIAN:
        lightNS_Red := TRUE;
        lightNS_Yellow := FALSE;
        lightNS_Green := FALSE;
        lightEW_Red := FALSE;
        lightEW_Yellow := FALSE;
        lightEW_Green := TRUE;
        crosswalkLight := TRUE;

        trafficTimer(IN := TRUE, PT := T#10s);

        IF trafficTimer.Q THEN
            trafficTimer(IN := FALSE);
            nextState := E_TrafficState.NORMAL;
        END_IF;

    // --- STATE: EMERGENCY (Override all other logic) ---
    E_TrafficState.EMERGENCY:
        lightNS_Red := TRUE;
        lightNS_Yellow := FALSE;
        lightNS_Green := FALSE;
        lightEW_Red := TRUE;
        lightEW_Yellow := FALSE;
        lightEW_Green := FALSE;
        crosswalkLight := FALSE;

        emergencyHoldTime(IN := NOT emergencyVehicleDetected, PT := T#5s);

        IF emergencyHoldTime.Q THEN
            emergencyHoldTime(IN := FALSE);
            nextState := E_TrafficState.NORMAL;
        END_IF;

END_CASE;

// Update current state
currentState := nextState;
