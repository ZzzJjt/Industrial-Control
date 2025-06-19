(* Traffic Light Control in IEC 61131-3 Structured Text *)
(* Purpose: Manage traffic light sequence with pedestrian and emergency vehicle priority *)

PROGRAM TrafficLightControl
VAR
    (* Inputs *)
    PedestrianRequest : BOOL;        (* TRUE when pedestrian push button is pressed *)
    EmergencyDetected : BOOL;        (* TRUE when emergency vehicle is detected *)

    (* Outputs *)
    RedLight : BOOL;                (* TRUE to activate red light *)
    GreenLight : BOOL;              (* TRUE to activate green light *)
    YellowLight : BOOL;             (* TRUE to activate yellow light *)
    WalkSignal : BOOL;              (* TRUE to activate pedestrian walk signal *)

    (* Internal Variables *)
    TrafficState : INT := 0;        (* 0=Red, 1=Green, 2=Yellow, 3=Pedestrian *)
    StateTimer : TON;               (* Timer for state transitions *)
    PedestrianActive : BOOL;        (* TRUE when pedestrian crossing is active *)
END_VAR

(* Main Control Logic *)
(* 1. Emergency Vehicle Override *)
IF EmergencyDetected THEN
    TrafficState := 1;          (* Force Green for emergency vehicle *)
    RedLight := FALSE;
    GreenLight := TRUE;
    YellowLight := FALSE;
    WalkSignal := FALSE;
    StateTimer(IN := FALSE);    (* Cancel any active timers *)
    PedestrianActive := FALSE;
ELSE
    (* 2. Normal Traffic Light Sequence with Pedestrian Handling *)
    CASE TrafficState OF
        0: (* Red State *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            WalkSignal := PedestrianActive;
            StateTimer(IN := TRUE, PT := T#10s);  (* Red for 10 seconds *)
            IF StateTimer.Q THEN
                IF PedestrianRequest AND NOT PedestrianActive THEN
                    PedestrianActive := TRUE;      (* Activate pedestrian crossing *)
                    TrafficState := 3;             (* Move to Pedestrian state *)
                    StateTimer(IN := FALSE);
                ELSE
                    TrafficState := 1;             (* Move to Green *)
                    StateTimer(IN := FALSE);
                END_IF;
            END_IF;

        1: (* Green State *)
            RedLight := FALSE;
            GreenLight := TRUE;
            YellowLight := FALSE;
            WalkSignal := FALSE;
            StateTimer(IN := TRUE, PT := T#10s);  (* Green for 10 seconds *)
            IF StateTimer.Q THEN
                TrafficState := 2;                (* Move to Yellow *)
                StateTimer(IN := FALSE);
            END_IF;

        2: (* Yellow State *)
            RedLight := FALSE;
            GreenLight := FALSE;
            YellowLight := TRUE;
            WalkSignal := FALSE;
            StateTimer(IN := TRUE, PT := T#3s);   (* Yellow for 3 seconds *)
            IF StateTimer.Q THEN
                TrafficState := 0;                (* Return to Red *)
                PedestrianActive := FALSE;
                StateTimer(IN := FALSE);
            END_IF;

        3: (* Pedestrian State *)
            RedLight := TRUE;
            GreenLight := FALSE;
            YellowLight := FALSE;
            WalkSignal := TRUE;
            StateTimer(IN := TRUE, PT := T#8s);   (* Pedestrian crossing for 8 seconds *)
            IF StateTimer.Q THEN
                TrafficState := 1;                (* Move to Green *)
                PedestrianRequest := FALSE;        (* Clear pedestrian request *)
                PedestrianActive := FALSE;
                StateTimer(IN := FALSE);
            END_IF;
    END_CASE;
END_IF;

(* Notes:
   - Emergency Priority:
     - EmergencyDetected forces GreenLight ON, overriding all other states
     - Clears timers and pedestrian signals for immediate response
   - Normal Sequence:
     - Red (10s) → Green (10s) → Yellow (3s) → Red
     - Pedestrian crossing (8s) inserted during Red when requested
   - Pedestrian Handling:
     - PedestrianRequest triggers WalkSignal during Red or Pedestrian state
     - PedestrianActive ensures one crossing per request
   - Safety:
     - RedLight ON during pedestrian crossing
     - No conflicting light states (only one light active at a time)
   - Physical Integration:
     - Inputs: PedestrianRequest (push button), EmergencyDetected (e.g., RFID or siren sensor)
     - Outputs: Relays for RedLight, GreenLight, YellowLight, WalkSignal
   - Scalability:
     - Add more directions by extending states
     - Adjust timer durations (PT) for different traffic patterns
   - Maintenance:
     - Integrate with HMI to display TrafficState, timer status, and requests
     - Log EmergencyDetected events for analysis
*)
END_PROGRAM
