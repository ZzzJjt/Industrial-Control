// === Traffic Light FSM in IEC 61131-3 ST ===

TYPE TrafficState : (IDLE, GREEN, YELLOW, RED, PEDESTRIAN_WAIT, EMERGENCY);
END_TYPE

VAR
    // Inputs
    pedestrianButtonPressed : BOOL;
    emergencyVehicleApproaching : BOOL;

    // Outputs
    greenLightOn : BOOL;
    yellowLightOn : BOOL;
    redLightOn : BOOL;

    // State
    currentState : TrafficState := IDLE;

    // Timer
    phaseTimer : TON;
    timerEnable : BOOL := FALSE;

    // Rising edge detection
    rPedestrian : R_TRIG;
    rEmergency : R_TRIG;

    // Internal
    tGreen  : TIME := T#10s;
    tYellow : TIME := T#3s;
    tRed    : TIME := T#7s;
END_VAR

// === Edge detection for rising input signals ===
rPedestrian(CLK := pedestrianButtonPressed);
rEmergency(CLK := emergencyVehicleApproaching);

// === Default: no timer unless activated below ===
timerEnable := FALSE;

// === State machine logic ===
CASE currentState OF

    IDLE:
        redLightOn := TRUE;
        greenLightOn := FALSE;
        yellowLightOn := FALSE;

        IF rEmergency.Q THEN
            currentState := EMERGENCY;
        ELSIF rPedestrian.Q THEN
            currentState := PEDESTRIAN_WAIT;
        ELSE
            currentState := GREEN;
            timerEnable := TRUE;
            phaseTimer(IN := timerEnable, PT := tGreen);
        END_IF

    GREEN:
        greenLightOn := TRUE;
        yellowLightOn := FALSE;
        redLightOn := FALSE;

        timerEnable := TRUE;
        phaseTimer(IN := timerEnable, PT := tGreen);

        IF phaseTimer.Q THEN
            currentState := YELLOW;
        END_IF

    YELLOW:
        greenLightOn := FALSE;
        yellowLightOn := TRUE;
        redLightOn := FALSE;

        timerEnable := TRUE;
        phaseTimer(IN := timerEnable, PT := tYellow);

        IF phaseTimer.Q THEN
            currentState := RED;
        END_IF

    RED:
        greenLightOn := FALSE;
        yellowLightOn := FALSE;
        redLightOn := TRUE;

        timerEnable := TRUE;
        phaseTimer(IN := timerEnable, PT := tRed);

        IF phaseTimer.Q THEN
            currentState := IDLE;
        END_IF

    PEDESTRIAN_WAIT:
        greenLightOn := FALSE;
        yellowLightOn := FALSE;
        redLightOn := TRUE;

        timerEnable := TRUE;
        phaseTimer(IN := timerEnable, PT := tRed);

        IF phaseTimer.Q THEN
            currentState := GREEN;
        END_IF

    EMERGENCY:
        greenLightOn := FALSE;
        yellowLightOn := FALSE;
        redLightOn := TRUE;

        // Stay in EMERGENCY until cleared
        IF NOT emergencyVehicleApproaching THEN
            currentState := IDLE;
        END_IF

END_CASE

// === Timer execution ===
phaseTimer(IN := timerEnable);
