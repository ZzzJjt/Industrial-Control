TYPE
    TRAFFIC_STATE : (
        GREEN,          (* Green light phase *)
        YELLOW,         (* Yellow light phase *)
        RED,            (* Red light phase *)
        PEDESTRIAN,     (* Pedestrian crossing phase *)
        EMERGENCY       (* Emergency vehicle override *)
    );
END_TYPE

VAR
    (* State variable *)
    Current_State   : TRAFFIC_STATE := GREEN;  (* Current traffic light state *)

    (* Inputs *)
    pedestrianButtonPressed   : BOOL;          (* Pedestrian button input *)
    emergencyVehicleApproaching : BOOL;       (* Emergency vehicle sensor input *)

    (* Outputs *)
    greenLightOn    : BOOL;                   (* Green light output *)
    yellowLightOn   : BOOL;                   (* Yellow light output *)
    redLightOn      : BOOL;                   (* Red light output *)

    (* Timers *)
    Phase_Timer     : TON;                    (* Timer for phase durations *)
    Green_Duration  : TIME := T#30s;          (* Green phase duration *)
    Yellow_Duration : TIME := T#5s;           (* Yellow phase duration *)
    Red_Duration    : TIME := T#20s;          (* Red phase duration *)
    Ped_Duration    : TIME := T#15s;          (* Pedestrian phase duration *)

    (* Edge detection *)
    Ped_Trigger     : R_TRIG;                 (* Edge detection for pedestrian button *)
    Emerg_Trigger   : R_TRIG;                 (* Edge detection for emergency vehicle *)
    Emerg_Clear     : F_TRIG;                 (* Edge detection for emergency clear *)

    (* Flags *)
    Ped_Request     : BOOL := FALSE;          (* Pedestrian request pending *)
END_VAR

(* Edge detection for inputs *)
Ped_Trigger(CLK := pedestrianButtonPressed);
Emerg_Trigger(CLK := emergencyVehicleApproaching);
Emerg_Clear(CLK := emergencyVehicleApproaching);

(* Finite State Machine *)
CASE Current_State OF
    GREEN:
        (* Set outputs *)
        greenLightOn := TRUE;
        yellowLightOn := FALSE;
        redLightOn := FALSE;

        (* Start timer *)
        Phase_Timer(IN := TRUE, PT := Green_Duration);

        (* Check for emergency or pedestrian request *)
        IF Emerg_Trigger.Q THEN
            Current_State := EMERGENCY;
            Phase_Timer(IN := FALSE);  (* Stop timer *)
            Ped_Request := FALSE;       (* Clear pending request *)
        ELSIF Ped_Trigger.Q THEN
            Ped_Request := TRUE;        (* Queue pedestrian request *)
        END_IF;

        (* Transition to yellow when timer expires *)
        IF Phase_Timer.Q AND NOT Emerg_Trigger.Q THEN
            Current_State := YELLOW;
            Phase_Timer(IN := FALSE);  (* Reset timer *)
        END_IF;

    YELLOW:
        (* Set outputs *)
        greenLightOn := FALSE;
        yellowLightOn := TRUE;
        redLightOn := FALSE;

        (* Start timer *)
        Phase_Timer(IN := TRUE, PT := Yellow_Duration);

        (* Check for emergency *)
        IF Emerg_Trigger.Q THEN
            Current_State := EMERGENCY;
            Phase_Timer(IN := FALSE);
            Ped_Request := FALSE;
        END_IF;

        (* Transition to red or pedestrian *)
        IF Phase_Timer.Q AND NOT Emerg_Trigger.Q THEN
            IF Ped_Request THEN
                Current_State := PEDESTRIAN;
                Ped_Request := FALSE;   (* Clear request *)
            ELSE
                Current_State := RED;
            END_IF;
            Phase_Timer(IN := FALSE);
        END_IF;

    RED:
        (* Set outputs *)
        greenLightOn := FALSE;
        yellowLightOn := FALSE;
        redLightOn := TRUE;

        (* Start timer *)
        Phase_Timer(IN := TRUE, PT := Red_Duration);

        (* Check for emergency or pedestrian request *)
        IF Emerg_Trigger.Q THEN
            Current_State := EMERGENCY;
            Phase_Timer(IN := FALSE);
            Ped_Request := FALSE;
        ELSIF Ped_Trigger.Q THEN
            Ped_Request := TRUE;
        END_IF;

        (* Transition to green *)
        IF Phase_Timer.Q AND NOT Emerg_Trigger.Q THEN
            Current_State := GREEN;
            Phase_Timer(IN := FALSE);
        END_IF;

    PEDESTRIAN:
        (* Set outputs *)
        greenLightOn := FALSE;
        yellowLightOn := FALSE;
        redLightOn := TRUE;  (* Red light stays on for vehicles *)

        (* Start timer *)
        Phase_Timer(IN := TRUE, PT := Ped_Duration);

        (* Check for emergency *)
        IF Emerg_Trigger.Q THEN
            Current_State := EMERGENCY;
            Phase_Timer(IN := FALSE);
            Ped_Request := FALSE;
        END_IF;

        (* Transition to red *)
        IF Phase_Timer.Q AND NOT Emerg_Trigger.Q THEN
            Current_State := RED;
            Phase_Timer(IN := FALSE);
        END_IF;

    EMERGENCY:
        (* Set outputs *)
        greenLightOn := TRUE;  (* Green light for emergency vehicle *)
        yellowLightOn := FALSE;
        redLightOn := FALSE;

        (* No timer; wait for emergency to clear *)
        IF Emerg_Clear.Q THEN
            Current_State := RED;  (* Return to red for safety *)
            Ped_Request := FALSE;  (* Clear any pending request *)
        END_IF;

END_CASE;

(* Reset timer if not in use *)
IF NOT Phase_Timer.IN THEN
    Phase_Timer.IN := FALSE;
END_IF;

(* Outputs are sent to traffic lights *)
(* Example: Write greenLightOn, yellowLightOn, redLightOn to digital outputs *)
