(* Program: Urea Reaction Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Automates urea synthesis by controlling valve operations, monitoring pressure and temperature, and managing reaction timing *)
PROGRAM UreaReactionControl
VAR
    (* Inputs *)
    AmmoniaValveState : BOOL;          (* TRUE if ammonia valve is open *)
    CO2ValveState : BOOL;             (* TRUE if CO2 valve is open *)
    Pressure_PV : REAL;               (* Current reactor pressure, bar *)
    Temperature_PV : REAL;            (* Current reactor temperature, °C *)
    EmergencyStop : BOOL;             (* TRUE if emergency stop activated *)
    
    (* Outputs *)
    AmmoniaValveCmd : BOOL;           (* TRUE to open ammonia valve *)
    CO2ValveCmd : BOOL;               (* TRUE to open CO2 valve *)
    ReactionComplete : BOOL;          (* TRUE when reaction is complete *)
    AlarmActive : BOOL;               (* TRUE if fault or emergency *)
    
    (* Internal Variables *)
    Step : UINT;                      (* Process step: 0=Idle, 1=Loading, 2=Reaction *)
    ReactionTimer : TON;              (* Timer for reaction duration *)
    StartTime : TIME;                 (* Reaction start time using CURRENT_TIME *)
    ValveOpenConfirmed : BOOL;        (* TRUE if both valves are open *)
    
    (* Parameters *)
    Pressure_SP : REAL := 150.0;      (* Target pressure, bar *)
    Pressure_Tol : REAL := 5.0;       (* Pressure tolerance, ±bar *)
    Temperature_SP : REAL := 185.0;   (* Target temperature, °C *)
    Temperature_Tol : REAL := 5.0;    (* Temperature tolerance, ±°C *)
    Reaction_Duration : TIME := T#30m; (* Reaction duration, 30 minutes *)
END_VAR

(* Initialize outputs and state *)
AmmoniaValveCmd := FALSE;             (* Ammonia valve closed *)
CO2ValveCmd := FALSE;                 (* CO2 valve closed *)
ReactionComplete := FALSE;            (* Reaction not complete *)
AlarmActive := FALSE;                 (* No initial alarm *)
Step := 0;                            (* Start in Idle *)
ReactionTimer(IN := FALSE, PT := Reaction_Duration); (* Initialize timer *)
ValveOpenConfirmed := FALSE;          (* Valves not confirmed open *)

(* Main logic *)
(* Emergency stop handling: Overrides all operations *)
IF EmergencyStop THEN
    (* Halt process and activate alarm *)
    AmmoniaValveCmd := FALSE;
    CO2ValveCmd := FALSE;
    ReactionComplete := FALSE;
    AlarmActive := TRUE;
    Step := 0;
    ReactionTimer(IN := FALSE);
    RETURN;
END_IF;

(* State machine *)
CASE Step OF
    0: (* Idle *)
        (* Reset outputs and wait for start *)
        AmmoniaValveCmd := FALSE;
        CO2ValveCmd := FALSE;
        ReactionComplete := FALSE;
        ReactionTimer(IN := FALSE);
        IF NOT AlarmActive THEN
            (* Start loading when conditions allow *)
            Step := 1;
        END_IF;

    1: (* Loading *)
        (* Open both valves *)
        AmmoniaValveCmd := TRUE;
        CO2ValveCmd := TRUE;
        (* Check valve states *)
        IF AmmoniaValveState AND CO2ValveState THEN
            ValveOpenConfirmed := TRUE;
            (* Record start time and proceed to reaction *)
            StartTime := CURRENT_TIME;
            Step := 2;
        END_IF;
        (* Timeout or fault check *)
        ReactionTimer(IN := TRUE, PT := T#10s); (* 10s timeout for valve opening *)
        IF ReactionTimer.Q AND NOT ValveOpenConfirmed THEN
            AlarmActive := TRUE;
            Step := 0;                    (* Return to Idle on failure *)
        END_IF;

    2: (* Reaction *)
        (* Monitor pressure and temperature *)
        IF Pressure_PV >= (Pressure_SP - Pressure_Tol) AND
           Pressure_PV <= (Pressure_SP + Pressure_Tol) AND
           Temperature_PV >= (Temperature_SP - Temperature_Tol) AND
           Temperature_PV <= (Temperature_SP + Temperature_Tol) THEN
            (* Conditions met: Start or continue reaction timer *)
            ReactionTimer(IN := TRUE);
            IF ReactionTimer.Q THEN
                (* Reaction complete *)
                ReactionComplete := TRUE;
                AmmoniaValveCmd := FALSE;
                CO2ValveCmd := FALSE;
                Step := 0;                (* Return to Idle *)
            END_IF;
        ELSE
            (* Conditions not met: Pause timer *)
            ReactionTimer(IN := FALSE);
        END_IF;
        (* Safety check for extreme conditions *)
        IF Pressure_PV > (Pressure_SP + 2.0 * Pressure_Tol) OR
           Pressure_PV < (Pressure_SP - 2.0 * Pressure_Tol) OR
           Temperature_PV > (Temperature_SP + 2.0 * Temperature_Tol) OR
           Temperature_PV < (Temperature_SP - 2.0 * Temperature_Tol) THEN
            AlarmActive := TRUE;
            AmmoniaValveCmd := FALSE;
            CO2ValveCmd := FALSE;
            Step := 0;                    (* Return to Idle *)
        END_IF;

END_CASE;

(* Ensure safe state on power-up or PLC stop *)
IF EmergencyStop OR AlarmActive THEN
    AmmoniaValveCmd := FALSE;
    CO2ValveCmd := FALSE;
    ReactionComplete := FALSE;
END_IF;

END_PROGRAM
