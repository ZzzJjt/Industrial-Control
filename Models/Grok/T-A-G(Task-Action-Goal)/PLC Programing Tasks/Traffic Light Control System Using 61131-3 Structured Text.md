(* Program: Traffic Light Control System *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Manages traffic light sequencing with pedestrian and emergency handling *)
(* Ensures safe transitions and real-time operation *)
PROGRAM PRG_TrafficLightControl
VAR
    (* Inputs *)
    PedestrianRequest : BOOL;          (* TRUE if pedestrian button pressed *)
    EmergencyDetected : BOOL;         (* TRUE if emergency vehicle detected *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    RedLight : BOOL;                  (* TRUE to activate red light *)
    GreenLight : BOOL;                (* TRUE to activate green light *)
    YellowLight : BOOL;               (* TRUE to activate yellow light *)
    AlarmActive : BOOL;               (* TRUE if emergency stop or error *)
    
    (* Internal Variables *)
    TrafficState : UINT;              (* State: 0=Red, 1=Green, 2=Yellow, 3=PedestrianRed *)
    StateTimer : TON;                 (* Timer for state durations *)
    LastPedestrianRequest : BOOL;     (* For rising edge detection on PedestrianRequest *)
    PedestrianActive : BOOL;          (* TRUE if pedestrian request being serviced *)
    LastEmergencyDetected : BOOL;     (* For edge detection on EmergencyDetected *)
    EmergencyActive : BOOL;           (* TRUE if emergency override active *)
END_VAR

(* Initialize outputs and state *)
RedLight := TRUE;                     (* Start with red light ON *)
GreenLight := FALSE;                  (* Green light OFF *)
YellowLight := FALSE;                 (* Yellow light OFF *)
AlarmActive := FALSE;                 (* No initial alarm *)
TrafficState := 0;                    (* Start in Red state *)
StateTimer(IN := FALSE, PT := T#10s); (* Initialize 10-second timer *)
PedestrianActive := FALSE;            (* No initial pedestrian request *)
EmergencyActive := FALSE;             (* No initial emergency override *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt system and activate alarm *)
    RedLight := TRUE;                 (* Red light ON: Stop traffic *)
    GreenLight := FALSE;              (* Green light OFF *)
    YellowLight := FALSE;             (* Yellow light OFF *)
    AlarmActive := TRUE;              (* Activate alarm *)
    StateTimer(IN := FALSE);          (* Reset timer *)
    TrafficState := 0;                (* Return to Red *)
    PedestrianActive := FALSE;        (* Clear pedestrian request *)
    EmergencyActive := FALSE;         (* Clear emergency override *)
    RETURN;
END_IF;

(* Emergency vehicle handling *)
(* Forces green light immediately when detected *)
IF EmergencyDetected AND NOT LastEmergencyDetected THEN
    (* Rising edge: Start emergency override *)
    EmergencyActive := TRUE;
    RedLight := FALSE;                (* Red light OFF *)
    GreenLight := TRUE;               (* Green light ON *)
    YellowLight := FALSE;             (* Yellow light OFF *)
    StateTimer(IN := FALSE);          (* Reset timer *)
    TrafficState := 1;                (* Set to Green *)
    PedestrianActive := FALSE;        (* Clear pedestrian request *)
ELSIF NOT EmergencyDetected AND EmergencyActive THEN
    (* Falling edge: Return to normal sequence *)
    EmergencyActive := FALSE;
    RedLight := TRUE;                 (* Red light ON: Safe default *)
    GreenLight := FALSE;              (* Green light OFF *)
    YellowLight := FALSE;             (* Yellow light OFF *)
    StateTimer(IN := FALSE);          (* Reset timer *)
    TrafficState := 0;                (* Return to Red *)
END_IF;

(* Update edge detection *)
LastEmergencyDetected := EmergencyDetected;

(* Normal operation: State machine *)
IF NOT EmergencyActive AND NOT EmergencyStop THEN
    (* Detect pedestrian request rising edge *)
    IF PedestrianRequest AND NOT LastPedestrianRequest AND TrafficState = 0 THEN
        PedestrianActive := TRUE;      (* Activate pedestrian request *)
    END_IF;
    LastPedestrianRequest := PedestrianRequest;

    CASE TrafficState OF
        0: (* Red *)
            (* Normal red phase: 10 seconds *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            IF PedestrianActive THEN
                (* Extend red for pedestrian request *)
                StateTimer(IN := TRUE, PT := T#15s); (* Extended to 15s *)
                IF StateTimer.Q THEN
                    (* Pedestrian request complete *)
                    PedestrianActive := FALSE;
                    PedestrianRequest := FALSE; (* Clear request *)
                    StateTimer(IN := FALSE);
                    TrafficState := 1;        (* Transition to Green *)
                END_IF;
            ELSE
                (* Normal red duration *)
                StateTimer(IN := TRUE, PT := T#10s);
                IF StateTimer.Q THEN
                    StateTimer(IN := FALSE);
                    TrafficState := 1;        (* Transition to Green *)
                END_IF;
            END_IF;

        1: (* Green *)
            (* Green phase: 10 seconds *)
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;
            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                TrafficState := 2;        (* Transition to Yellow *)
            END_IF;

        2: (* Yellow *)
            (* Yellow phase: 3 seconds *)
            RedLight := FALSE;
            GreenLight := FALSE;
            YellowLight := TRUE;
            StateTimer(IN := TRUE, PT := T#3s);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                TrafficState := 0;        (* Transition to Red *)
                PedestrianRequest := FALSE; (* Clear any pending request *)
                PedestrianActive := FALSE; (* Clear pedestrian state *)
            END_IF;

        3: (* Reserved for future expansion, e.g., PedestrianRed with different timing *)
            (* Fallback to Red *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            TrafficState := 0;
    END_CASE;
END_IF;

(* Ensure safe state on power-up or PLC stop *)
(* Red light ON if emergency or error *)
IF EmergencyStop THEN
    RedLight := TRUE;
    GreenLight := FALSE;
    YellowLight := FALSE;
END_IF;

END_PROGRAM
