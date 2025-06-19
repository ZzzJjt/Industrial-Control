PROGRAM TrafficLightControl
VAR
    (* Inputs *)
    PedestrianButton : BOOL; (* TRUE when pedestrian button is pressed *)
    EmergencyDetected : BOOL; (* TRUE when emergency vehicle is detected *)
    
    (* Outputs *)
    RedLight : BOOL; (* TRUE to activate red light *)
    GreenLight : BOOL; (* TRUE to activate green light *)
    YellowLight : BOOL; (* TRUE to activate yellow light *)
    
    (* Internal variables *)
    State : INT := 0; (* State machine: 0=NormalGreen, 1=NormalYellow, 2=NormalRed, 3=PedestrianRed, 4=EmergencyGreen *)
    PhaseTimer : TON; (* Timer for phase duration *)
    PedestrianTrigger : R_TRIG; (* Rising edge detection for pedestrian button *)
    EmergencyTrigger : R_TRIG; (* Rising edge detection for emergency detection *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid state *)
END_VAR

(* Rising edge detection *)
PedestrianTrigger(CLK := PedestrianButton);
EmergencyTrigger(CLK := EmergencyDetected);

(* State machine *)
CASE State OF
    0: (* Normal Green *)
        RedLight := FALSE;
        GreenLight := TRUE;
        YellowLight := FALSE;
        IF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE); (* Reset timer *)
            State := 1; (* Transition to Normal Yellow *)
        ELSIF EmergencyTrigger.Q THEN
            PhaseTimer(IN := FALSE);
            State := 4; (* Transition to Emergency Green *)
        ELSIF PedestrianTrigger.Q THEN
            PhaseTimer(IN := FALSE);
            State := 1; (* Transition to Normal Yellow to prepare for pedestrian *)
        ELSE
            PhaseTimer(IN := TRUE, PT := T#10s); (* 10-second green phase *)
        END_IF;
    
    1: (* Normal Yellow *)
        RedLight := FALSE;
        GreenLight := FALSE;
        YellowLight := TRUE;
        IF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE);
            IF PedestrianTrigger.Q THEN
                State := 3; (* Transition to Pedestrian Red *)
            ELSE
                State := 2; (* Transition to Normal Red *)
            END_IF;
        ELSIF EmergencyTrigger.Q THEN
            PhaseTimer(IN := FALSE);
            State := 4; (* Transition to Emergency Green *)
        ELSE
            PhaseTimer(IN := TRUE, PT := T#3s); (* 3-second yellow phase *)
        END_IF;
    
    2: (* Normal Red *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        IF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE);
            State := 0; (* Transition to Normal Green *)
        ELSIF EmergencyTrigger.Q THEN
            PhaseTimer(IN := FALSE);
            State := 4; (* Transition to Emergency Green *)
        ELSIF PedestrianTrigger.Q THEN
            PhaseTimer(IN := FALSE);
            State := 3; (* Transition to Pedestrian Red *)
        ELSE
            PhaseTimer(IN := TRUE, PT := T#10s); (* 10-second red phase *)
        END_IF;
    
    3: (* Pedestrian Red *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        IF PhaseTimer.Q THEN
            PhaseTimer(IN := FALSE);
            State := 0; (* Transition to Normal Green *)
        ELSIF EmergencyTrigger.Q THEN
            PhaseTimer(IN := FALSE);
            State := 4; (* Transition to Emergency Green *)
        ELSE
            PhaseTimer(IN := TRUE, PT := T#15s); (* 15-second pedestrian crossing *)
        END_IF;
    
    4: (* Emergency Green *)
        RedLight := FALSE;
        GreenLight := TRUE;
        YellowLight := FALSE;
        IF NOT EmergencyDetected THEN
            PhaseTimer(IN := FALSE);
            State := 2; (* Transition to Normal Red for safety *)
        ELSE
            PhaseTimer(IN := FALSE); (* No timer during emergency *)
        END_IF;
    
    ELSE
        (* Invalid state: reset to Normal Red *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        PhaseTimer(IN := FALSE);
        State := 2;
        ErrorCode := 1;
END_CASE;

END_PROGRAM
