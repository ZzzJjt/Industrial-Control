// Detect rising edges on inputs
RTrigPedestrian(CLK := PedestrianButtonPressed);
RTrigEmergency(CLK := EmergencyVehicleApproaching);

// If emergency detected, immediately switch to EMERGENCY state
IF RTrigEmergency.Q THEN
    CurrentState := E_TRAFFIC_LIGHT_STATE.EMERGENCY;
    StateTimer(IN := FALSE); // Reset timer
END_IF;

CASE CurrentState OF

    E_TRAFFIC_LIGHT_STATE.IDLE:
        RedLightOn := TRUE;
        GreenLightOn := FALSE;
        YellowLightOn := FALSE;

        IF NOT EmergencyVehicleApproaching THEN
            CurrentState := E_TRAFFIC_LIGHT_STATE.GREEN;
        END_IF;

    E_TRAFFIC_LIGHT_STATE.GREEN:
        RedLightOn := FALSE;
        GreenLightOn := TRUE;
        YellowLightOn := FALSE;

        // Start timer for green phase
        StateTimer(IN := TRUE, PT := T#30s);

        // Check for pedestrian request
        IF RTrigPedestrian.Q THEN
            PedestrianRequest := TRUE;
        END_IF;

        IF StateTimer.Q THEN
            StateTimer(IN := FALSE); // Stop timer
            CurrentState := E_TRAFFIC_LIGHT_STATE.YELLOW;
        END_IF;

    E_TRAFFIC_LIGHT_STATE.YELLOW:
        RedLightOn := FALSE;
        GreenLightOn := FALSE;
        YellowLightOn := TRUE;

        StateTimer(IN := TRUE, PT := T#5s);

        IF StateTimer.Q THEN
            StateTimer(IN := FALSE);
            CurrentState := E_TRAFFIC_LIGHT_STATE.RED;
        END_IF;

    E_TRAFFIC_LIGHT_STATE.RED:
        RedLightOn := TRUE;
        GreenLightOn := FALSE;
        YellowLightOn := FALSE;

        IF PedestrianRequest THEN
            CurrentState := E_TRAFFIC_LIGHT_STATE.PEDESTRIAN_WAIT;
            PedestrianRequest := FALSE;
        ELSE
            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                CurrentState := E_TRAFFIC_LIGHT_STATE.GREEN;
            END_IF;
        END_IF;

    E_TRAFFIC_LIGHT_STATE.PEDESTRIAN_WAIT:
        RedLightOn := TRUE;
        GreenLightOn := FALSE;
        YellowLightOn := FALSE;

        StateTimer(IN := TRUE, PT := T#20s); // Safe walk time

        IF StateTimer.Q THEN
            StateTimer(IN := FALSE);
            CurrentState := E_TRAFFIC_LIGHT_STATE.GREEN;
        END_IF;

    E_TRAFFIC_LIGHT_STATE.EMERGENCY:
        RedLightOn := FALSE;
        GreenLightOn := FALSE;
        YellowLightOn := TRUE;

        // Flash yellow rapidly (handled via external flashing logic or HMI)
        StateTimer(IN := TRUE, PT := T#500ms);
        IF StateTimer.Q THEN
            YellowLightOn := NOT YellowLightOn;
            StateTimer(IN := FALSE);
        END_IF;

        // Exit emergency when signal cleared
        IF NOT EmergencyVehicleApproaching THEN
            CurrentState := E_TRAFFIC_LIGHT_STATE.IDLE;
        END_IF;

END_CASE;
