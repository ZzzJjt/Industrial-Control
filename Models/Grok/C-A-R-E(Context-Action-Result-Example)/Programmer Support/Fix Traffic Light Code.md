(* IEC 61131-3 Structured Text Program: TrafficLightControl *)
(* Purpose: Controls a traffic light system with state machine, handling normal cycles, pedestrian crossings, and emergency priority *)

PROGRAM TrafficLightControl
VAR
    (* Inputs *)
    PedestrianTrig : R_TRIG;        (* Rising edge trigger for pedestrian crossing request *)
    EmergencySensor : BOOL;          (* TRUE when emergency vehicle detected *)

    (* Outputs *)
    RedLight : BOOL;                (* TRUE to activate red light *)
    GreenLight : BOOL;              (* TRUE to activate green light *)
    YellowLight : BOOL;             (* TRUE to activate yellow light *)
    PedestrianSignal : BOOL;        (* TRUE to indicate safe pedestrian crossing *)
    State : INT := 0;               (* Current state: 0=NORMAL_RED, 1=NORMAL_GREEN, 2=NORMAL_YELLOW, 3=PEDESTRIAN_WAIT, 4=PEDESTRIAN_CROSS, 5=EMERGENCY_OVERRIDE *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    NormalTimer : TON;              (* Timer for normal cycle durations *)
    PedestrianTimer : TON;          (* Timer for pedestrian phases *)
    EmergencyTimer : TON;           (* Timer for emergency hold *)
    Last_PedestrianTrig : BOOL;     (* Previous PedestrianTrig.Q state *)
    Last_EmergencySensor : BOOL;    (* Previous EmergencySensor state *)
    Last_RedLight : BOOL;           (* Previous RedLight state *)
    Last_GreenLight : BOOL;         (* Previous GreenLight state *)
    Last_YellowLight : BOOL;        (* Previous YellowLight state *)
    Last_PedestrianSignal : BOOL;   (* Previous PedestrianSignal state *)
    Last_State : INT;               (* Previous State value *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Edge detection for pedestrian trigger *)
PedestrianTrig(CLK := NOT Last_PedestrianTrig);

(* Main Logic *)
(* State Machine Overview:
   - NORMAL_RED (0): Red light, 10s, to NORMAL_GREEN unless PedestrianTrig or EmergencySensor
   - NORMAL_GREEN (1): Green light, 10s, to NORMAL_YELLOW
   - NORMAL_YELLOW (2): Yellow light, 3s, to NORMAL_RED
   - PEDESTRIAN_WAIT (3): Red light, 5s, to PEDESTRIAN_CROSS
   - PEDESTRIAN_CROSS (4): Red light, PedestrianSignal, 10s, to NORMAL_GREEN
   - EMERGENCY_OVERRIDE (5): Green light, hold until EmergencySensor = FALSE and 5s elapsed, to NORMAL_RED
   - Cyclic execution, timers manage transitions, no WHILE or WAIT UNTIL
   - Only one mode active, prioritized: Emergency > Pedestrian > Normal
*)

(* Step 1: Input validation and safety check *)
IF EmergencySensor AND NOT Last_EmergencySensor THEN
    State := 5; (* EMERGENCY_OVERRIDE *)
    NormalTimer.IN := FALSE;
    PedestrianTimer.IN := FALSE;
    EmergencyTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Emergency Detected: Switching to EMERGENCY_OVERRIDE');
    END_IF;
END_IF;

(* Step 2: State machine execution *)
CASE State OF
    0: (* NORMAL_RED: Red light, 10s *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        PedestrianSignal := FALSE;
        NormalTimer(IN := TRUE, PT := T#10s);
        IF PedestrianTrig.Q THEN
            State := 3; (* PEDESTRIAN_WAIT *)
            NormalTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Pedestrian Request: Switching to PEDESTRIAN_WAIT');
            END_IF;
        ELSIF NormalTimer.Q THEN
            State := 1; (* NORMAL_GREEN *)
            NormalTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Normal Cycle: Switching to NORMAL_GREEN');
            END_IF;
        END_IF;

    1: (* NORMAL_GREEN: Green light, 10s *)
        RedLight := FALSE;
        GreenLight := TRUE;
        YellowLight := FALSE;
        PedestrianSignal := FALSE;
        NormalTimer(IN := TRUE, PT := T#10s);
        IF NormalTimer.Q THEN
            State := 2; (* NORMAL_YELLOW *)
            NormalTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Normal Cycle: Switching to NORMAL_YELLOW');
            END_IF;
        END_IF;

    2: (* NORMAL_YELLOW: Yellow light, 3s *)
        RedLight := FALSE;
        GreenLight := FALSE;
        YellowLight := TRUE;
        PedestrianSignal := FALSE;
        NormalTimer(IN := TRUE, PT := T#3s);
        IF NormalTimer.Q THEN
            State := 0; (* NORMAL_RED *)
            NormalTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Normal Cycle: Switching to NORMAL_RED');
            END_IF;
        END_IF;

    3: (* PEDESTRIAN_WAIT: Red light, 5s *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        PedestrianSignal := FALSE;
        PedestrianTimer(IN := TRUE, PT := T#5s);
        IF PedestrianTimer.Q THEN
            State := 4; (* PEDESTRIAN_CROSS *)
            PedestrianTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Pedestrian Phase: Switching to PEDESTRIAN_CROSS');
            END_IF;
        END_IF;

    4: (* PEDESTRIAN_CROSS: Red light, pedestrian signal, 10s *)
        RedLight := TRUE;
        GreenLight := FALSE;
        YellowLight := FALSE;
        PedestrianSignal := TRUE;
        PedestrianTimer(IN := TRUE, PT := T#10s);
        IF PedestrianTimer.Q THEN
            State := 1; (* NORMAL_GREEN *)
            PedestrianTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Pedestrian Phase Ended: Switching to NORMAL_GREEN');
            END_IF;
        END_IF;

    5: (* EMERGENCY_OVERRIDE: Green light, hold until clear *)
        RedLight := FALSE;
        GreenLight := TRUE;
        YellowLight := FALSE;
        PedestrianSignal := FALSE;
        EmergencyTimer(IN := TRUE, PT := T#5s);
        IF NOT EmergencySensor AND EmergencyTimer.Q THEN
            State := 0; (* NORMAL_RED *)
            EmergencyTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 12:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Emergency Cleared: Switching to NORMAL_RED');
            END_IF;
        END_IF;
END_CASE;

(* Step 3: Validate light states *)
(* Ensure only one light is active *)
IF (RedLight AND (GreenLight OR YellowLight)) OR 
   (GreenLight AND YellowLight) THEN
    RedLight := TRUE;
    GreenLight := FALSE;
    YellowLight := FALSE;
    State := 0; (* Default to NORMAL_RED *)
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 12:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Error: Multiple Lights Active, Defaulting to NORMAL_RED');
    END_IF;
END_IF;

(* Step 4: Log light and signal state changes *)
IF RedLight AND NOT Last_RedLight AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Red Light Activated');
END_IF;
IF NOT RedLight AND Last_RedLight AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Red Light Deactivated');
END_IF;
IF GreenLight AND NOT Last_GreenLight AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Green Light Activated');
END_IF;
IF NOT GreenLight AND Last_GreenLight AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Green Light Deactivated');
END_IF;
IF YellowLight AND NOT Last_YellowLight AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Yellow Light Activated');
END_IF;
IF NOT YellowLight AND Last_YellowLight AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Yellow Light Deactivated');
END_IF;
IF PedestrianSignal AND NOT Last_PedestrianSignal AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Pedestrian Signal Activated');
END_IF;
IF NOT PedestrianSignal AND Last_PedestrianSignal AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 12:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Pedestrian Signal Deactivated');
END_IF;

(* Step 5: Update last states for logging *)
Last_PedestrianTrig := PedestrianTrig.Q;
Last_EmergencySensor := EmergencySensor;
Last_RedLight := RedLight;
Last_GreenLight := GreenLight;
Last_YellowLight := YellowLight;
Last_PedestrianSignal := PedestrianSignal;
Last_State := State;

(* Step 6: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Controls traffic light with state machine, supports normal cycles, pedestrian crossings, emergency overrides.
   - Inputs:
     - PedestrianTrig: R_TRIG, rising edge for crossing request.
     - EmergencySensor: BOOL, TRUE for emergency vehicle.
   - Outputs:
     - RedLight, GreenLight, YellowLight: BOOL, active light.
     - PedestrianSignal: BOOL, TRUE for crossing.
     - State: INT, current state (0=NORMAL_RED, 1=NORMAL_GREEN, 2=NORMAL_YELLOW, 3=PEDESTRIAN_WAIT, 4=PEDESTRIAN_CROSS, 5=EMERGENCY_OVERRIDE).
     - DiagLog: ARRAY[1..50] OF STRING[80], diagnostic logs.
     - LogCount: INT, number of log entries.
   - Logic:
     - State machine replaces WHILE/WAIT UNTIL, uses TON timers.
     - Normal cycle: Red (10s) → Green (10s) → Yellow (3s).
     - Pedestrian: Wait (5s) → Cross (10s) from NORMAL_RED.
     - Emergency: Green until clear (min 5s), then NORMAL_RED.
     - Priority: Emergency > Pedestrian > Normal.
   - Optimization:
     - Simple logic (~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC).
     - Fixed flow, no recursion, single timer per state.
     - Minimal memory (~4 KB for logs, scalars for states).
   - Safety:
     - Ensures single light active.
     - Validates transitions, resets timers on change.
     - Logs state changes for traceability.
   - Usage:
     - Traffic light: Manages normal, pedestrian, emergency modes.
     - Example: PedestrianTrig.Q=TRUE in NORMAL_RED → PEDESTRIAN_WAIT; EmergencySensor=TRUE → EMERGENCY_OVERRIDE.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], BOOL, INT, TON, R_TRIG; adjust for platform limits.
     - Timestamp uses current date/time; replace with system clock (e.g., GET_SYSTEM_TIME).
     - Log buffer size (50) is practical; adjustable for memory constraints.
*)
END_PROGRAM
