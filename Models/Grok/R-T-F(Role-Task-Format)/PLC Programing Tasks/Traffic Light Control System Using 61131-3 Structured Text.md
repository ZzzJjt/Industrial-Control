PROGRAM TrafficLightControl
VAR
    (* Inputs *)
    PedestrianRequest : BOOL; (* TRUE when pedestrian push button is pressed *)
    EmergencyDetected : BOOL; (* TRUE when emergency vehicle is detected *)
    
    (* Outputs *)
    RedLight : BOOL; (* TRUE to activate red light *)
    GreenLight : BOOL; (* TRUE to activate green light *)
    YellowLight : BOOL; (* TRUE to activate yellow light *)
    
    (* Internal variables *)
    TrafficState : INT := 0; (* State: 0=Red, 1=Green, 2=Yellow, 3=PedestrianRed *)
    StateTimer : TON; (* Timer for state duration *)
    PedestrianActive : BOOL := FALSE; (* TRUE during pedestrian crossing *)
    PrevPedestrianRequest : BOOL; (* Tracks previous pedestrian request state *)
    ErrorCode : INT := 0; (* 0: Success, 1: Invalid state *)
END_VAR

(* Initialize outputs *)
RedLight := TRUE; (* Start with red for safety *)
GreenLight := FALSE;
YellowLight := FALSE;

(* Detect rising edge of PedestrianRequest *)
IF PedestrianRequest AND NOT PrevPedestrianRequest THEN
    PedestrianActive := TRUE;
END_IF;

(* Emergency vehicle priority *)
IF EmergencyDetected THEN
    (* Override: force green light *)
    TrafficState := 1;
    RedLight := FALSE;
    GreenLight := TRUE;
    YellowLight := FALSE;
    StateTimer(IN := FALSE); (* Reset timer *)
    PedestrianActive := FALSE; (* Cancel pedestrian crossing *)
ELSE
    (* Normal operation *)
    CASE TrafficState OF
        0: (* Red *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            StateTimer(IN := TRUE, PT := T#10s); (* 10-second red *)
            IF StateTimer.Q THEN
                IF PedestrianActive THEN
                    TrafficState := 3; (* Extend red for pedestrians *)
                    StateTimer(IN := FALSE);
                ELSE
                    TrafficState := 1; (* Proceed to green *)
                    StateTimer(IN := FALSE);
                END_IF;
            END_IF;
        
        1: (* Green *)
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;
            StateTimer(IN := TRUE, PT := T#10s); (* 10-second green *)
            IF StateTimer.Q THEN
                TrafficState := 2; (* Proceed to yellow *)
                StateTimer(IN := FALSE);
            END_IF;
        
        2: (* Yellow *)
            RedLight := FALSE;
            GreenLight := FALSE;
            YellowLight := TRUE;
            StateTimer(IN := TRUE, PT := T#3s); (* 3-second yellow *)
            IF StateTimer.Q THEN
                TrafficState := 0; (* Return to red *)
                PedestrianActive := FALSE; (* Clear pedestrian crossing *)
                StateTimer(IN := FALSE);
            END_IF;
        
        3: (* Pedestrian Red *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            StateTimer(IN := TRUE, PT := T#15s); (* 15-second pedestrian crossing *)
            IF StateTimer.Q THEN
                TrafficState := 1; (* Proceed to green *)
                PedestrianActive := FALSE; (* Clear pedestrian crossing *)
                StateTimer(IN := FALSE);
            END_IF;
        
        ELSE
            (* Invalid state: reset to red *)
            TrafficState := 0;
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            StateTimer(IN := FALSE);
            PedestrianActive := FALSE;
            ErrorCode := 1;
    END_CASE;
END_IF;

(* Update previous pedestrian request state *)
PrevPedestrianRequest := PedestrianRequest;

END_PROGRAM
