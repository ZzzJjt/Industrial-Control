(* Program: Traffic Light Control System *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Manages traffic lights with normal, pedestrian, and emergency conditions *)
(* Uses FSM for cyclic PLC execution, compliant with IEC 61131-3 *)
PROGRAM PRG_TrafficLightControl
VAR
    (* Inputs *)
    PedestrianRequest : BOOL;          (* TRUE if pedestrian button pressed *)
    EmergencyDetected : BOOL;          (* TRUE if emergency vehicle detected *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    RedLight : BOOL;                  (* TRUE to activate red light *)
    GreenLight : BOOL;                (* TRUE to activate green light *)
    YellowLight : BOOL;               (* TRUE to activate yellow light *)
    AlarmActive : BOOL;               (* TRUE if emergency stop or error *)
    
    (* Internal Variables *)
    State : UINT;                     (* FSM states: 0=Green, 1=Yellow, 2=Red, 3=PedestrianRed, 4=EmergencyGreen *)
    StateTimer : TON;                 (* Timer for state durations *)
    PedTrig : R_TRIG;                 (* Rising edge detection for PedestrianRequest *)
    EmergTrig : R_TRIG;               (* Rising edge detection for EmergencyDetected *)
    PedestrianPending : BOOL;         (* TRUE if pedestrian request is active *)
    EmergencyActive : BOOL;           (* TRUE if emergency override active *)
END_VAR

(* Initialize outputs and state *)
RedLight := FALSE;                    (* Red light OFF *)
GreenLight := TRUE;                   (* Start with green light ON *)
YellowLight := FALSE;                 (* Yellow light OFF *)
AlarmActive := FALSE;                 (* No initial alarm *)
State := 0;                           (* Start in Green state *)
StateTimer(IN := FALSE, PT := T#10s); (* Initialize timer *)
PedestrianPending := FALSE;           (* No initial pedestrian request *)
EmergencyActive := FALSE;             (* No initial emergency override *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt system and activate alarm *)
    RedLight := TRUE;
    GreenLight := FALSE;
    YellowLight := FALSE;
    AlarmActive := TRUE;
    StateTimer(IN := FALSE);
    State := 2;                       (* Force Red state *)
    PedestrianPending := FALSE;
    EmergencyActive := FALSE;
    RETURN;
END_IF;

(* Rising edge detection *)
PedTrig(CLK := PedestrianRequest);    (* Detect new pedestrian request *)
EmergTrig(CLK := EmergencyDetected);  (* Detect new emergency vehicle *)

(* State machine *)
IF NOT EmergencyStop THEN
    (* Handle emergency vehicle detection *)
    IF EmergTrig.Q THEN
        (* New emergency detected: Override to EmergencyGreen *)
        EmergencyActive := TRUE;
        StateTimer(IN := FALSE);
        State := 4;                   (* Switch to EmergencyGreen *)
    ELSIF EmergencyActive AND NOT EmergencyDetected THEN
        (* Emergency cleared: Return to Red *)
        EmergencyActive := FALSE;
        StateTimer(IN := FALSE);
        State := 2;                   (* Safe default to Red *)
    END_IF;

    (* Process pedestrian request *)
    IF PedTrig.Q AND State = 2 AND NOT EmergencyActive THEN
        (* New pedestrian request in Red state *)
        PedestrianPending := TRUE;
    END_IF;

    (* State transitions *)
    CASE State OF
        0: (* Green *)
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;
            StateTimer(IN := TRUE, PT := T#10s);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                State := 1;               (* Transition to Yellow *)
            END_IF;

        1: (* Yellow *)
            RedLight := FALSE;
            GreenLight := FALSE;
            YellowLight := TRUE;
            StateTimer(IN := TRUE, PT := T#3s);
            IF StateTimer.Q THEN
                StateTimer(IN := FALSE);
                State := 2;               (* Transition to Red *)
            END_IF;

        2: (* Red *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            IF PedestrianPending THEN
                StateTimer(IN := TRUE, PT := T#15s); (* Extended for pedestrian *)
                IF StateTimer.Q THEN
                    PedestrianPending := FALSE; (* Clear request *)
                    StateTimer(IN := FALSE);
                    State := 0;           (* Transition to Green *)
                END_IF;
            ELSE
                StateTimer(IN := TRUE, PT := T#10s);
                IF StateTimer.Q THEN
                    StateTimer(IN := FALSE);
                    State := 0;           (* Transition to Green *)
                END_IF;
            END_IF;

        3: (* Reserved for future expansion *)
            (* Fallback to Red *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            StateTimer(IN := FALSE);
            State := 2;

        4: (* EmergencyGreen *)
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;
            (* Remain in EmergencyGreen until EmergencyDetected clears *)
            StateTimer(IN := FALSE);      (* Timer not used *)
    END_CASE;
END_IF;

(* Ensure safe state on power-up or PLC stop *)
IF EmergencyStop THEN
    RedLight := TRUE;
    GreenLight := FALSE;
    YellowLight := FALSE;
END_IF;

END_PROGRAM
