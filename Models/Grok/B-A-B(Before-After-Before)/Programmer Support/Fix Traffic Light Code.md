(* Fixed Traffic Light Control in IEC 61131-3 Structured Text *)
(* Purpose: Safe and predictable control with state machine, proper timer usage, and exclusive modes *)

PROGRAM TrafficLightControlFixed
VAR
    (* Inputs *)
    PedestrianButton : BOOL;         (* TRUE when pedestrian button is pressed *)
    EmergencyDetected : BOOL;        (* TRUE when emergency vehicle is detected *)

    (* Outputs *)
    RedLight : BOOL;                (* TRUE to activate red light *)
    GreenLight : BOOL;              (* TRUE to activate green light *)
    YellowLight : BOOL;             (* TRUE to activate yellow light *)
    WalkSignal : BOOL;              (* TRUE to activate pedestrian walk signal *)

    (* State Machine *)
    TrafficState : INT;              (* 0=Red, 1=Green, 2=Yellow, 3=Pedestrian, 4=Emergency *)
    CONSTANT
        STATE_RED : INT := 0;
        STATE_GREEN : INT := 1;
        STATE_YELLOW : INT := 2;
        STATE_PEDESTRIAN : INT := 3;
        STATE_EMERGENCY : INT := 4;
    END_CONSTANT

    (* Internal Variables *)
    StateTimer : TON;               (* Timer for state transitions *)
    PedestrianRequest : BOOL;       (* Latched pedestrian request *)
    Ped_Button_Trigger : R_TRIG;    (* Rising edge detection for pedestrian button *)

    (* Timing Parameters *)
    Red_Duration : TIME := T#10s;   (* Red light duration *)
    Green_Duration : TIME := T#10s; (* Green light duration *)
    Yellow_Duration : TIME := T#3s; (* Yellow light duration *)
    Pedestrian_Duration : TIME := T#8s; (* Pedestrian crossing duration *)
END_VAR

(* Edge Detection for Pedestrian Button *)
Ped_Button_Trigger(CLK := PedestrianButton);
IF Ped_Button_Trigger.Q THEN
    PedestrianRequest := TRUE;   (* Latch pedestrian request on rising edge *)
END_IF;

(* Main Control Logic *)
CASE TrafficState OF
    STATE_EMERGENCY:
        (* Emergency Mode: Prioritize emergency vehicle *)
        RedLight := FALSE;
        GreenLight := TRUE;
        YellowLight := FALSE;
        WalkSignal := FALSE;
        StateTimer(IN := FALSE);    (* No timer in emergency mode *)
        IF NOT EmergencyDetected THEN
            TrafficState := STATE_RED;  (* Return to Red after emergency *)
            PedestrianRequest := FALSE; (* Clear pedestrian request *)
        END_IF;

    STATE_RED:
        (* Red State: Allow pedestrian crossing if requested *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        WalkSignal := PedestrianRequest;
        StateTimer(IN := TRUE, PT := Red_Duration);
        IF StateTimer.Q THEN
            IF PedestrianRequest THEN
                TrafficState := STATE_PEDESTRIAN; (* Transition to Pedestrian *)
                StateTimer(IN := FALSE);
            ELSE
                TrafficState := STATE_GREEN;      (* Transition to Green *)
                StateTimer(IN := FALSE);
            END_IF;
        END_IF;

    STATE_GREEN:
        (* Green State: Normal traffic flow *)
        RedLight := FALSE;
        GreenLight := TRUE;
        YellowLight := FALSE;
        WalkSignal := FALSE;
        StateTimer(IN := TRUE, PT := Green_Duration);
        IF StateTimer.Q THEN
            TrafficState := STATE_YELLOW;         (* Transition to Yellow *)
            StateTimer(IN := FALSE);
        END_IF;

    STATE_YELLOW:
        (* Yellow State: Transition phase *)
        RedLight := FALSE;
        GreenLight := FALSE;
        YellowLight := TRUE;
        WalkSignal := FALSE;
        StateTimer(IN := TRUE, PT := Yellow_Duration);
        IF StateTimer.Q THEN
            TrafficState := STATE_RED;            (* Return to Red *)
            StateTimer(IN := FALSE);
        END_IF;

    STATE_PEDESTRIAN:
        (* Pedestrian State: Safe crossing *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        WalkSignal := TRUE;
        StateTimer(IN := TRUE, PT := Pedestrian_Duration);
        IF StateTimer.Q THEN
            TrafficState := STATE_GREEN;          (* Return to Green *)
            PedestrianRequest := FALSE;           (* Clear pedestrian request *)
            StateTimer(IN := FALSE);
        END_IF;

ELSE
    (* Default: Initialize to Red *)
    TrafficState := STATE_RED;
    RedLight := TRUE;
    GreenLight := FALSE;
    YellowLight := FALSE;
    WalkSignal := FALSE;
    StateTimer(IN := FALSE);
END_CASE;

(* Emergency Override *)
IF EmergencyDetected THEN
    TrafficState := STATE_EMERGENCY;
    StateTimer(IN := FALSE);
END_IF;

(* Notes:
   - Fixes Applied:
     - Removed WHILE TRUE and WAIT UNTIL (non-compliant in cyclic PLCs)
     - Implemented state machine for clear, predictable control flow
     - Proper TON timer management: .IN toggled only at state transitions
     - R_TRIG for edge detection on PedestrianButton to latch requests
     - Avoids multiple light state updates in one cycle
   - Safety:
     - Emergency mode is mutually exclusive, overrides all states
     - Pedestrian mode only during Red or Pedestrian state
     - Single light state active at a time to prevent overlaps
   - Timer Usage:
     - StateTimer.IN set TRUE at state entry, FALSE at exit
     - .Q and .ET used for state transitions
   - Physical Integration:
     - Inputs: PedestrianButton (push button), EmergencyDetected (e.g., RFID sensor)
     - Outputs: Relays for RedLight, GreenLight, YellowLight, WalkSignal
   - Scalability:
     - Adjust timing (Red_Duration, etc.) for different intersections
     - Add states for multi-direction control
   - Maintenance:
     - Add HMI to display TrafficState, StateTimer.ET, PedestrianRequest
     - Log EmergencyDetected events for analysis
*)
END_PROGRAM
