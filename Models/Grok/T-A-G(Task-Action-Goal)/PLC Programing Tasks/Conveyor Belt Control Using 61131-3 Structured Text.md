(* Program: Conveyor Belt Control *)
(* Version: 1.0, Date: 2025-05-21 *)
(* Controls conveyor with manual stops and object sensors in auto/manual modes *)
(* Maintains constant speed of 2 m/s, prioritizes safety *)
PROGRAM PRG_ConveyorControl
VAR
    (* Inputs *)
    StationStop1 : BOOL;              (* Manual stop station 1: TRUE = Stop *)
    StationStop2 : BOOL;              (* Manual stop station 2: TRUE = Stop *)
    StationStop3 : BOOL;              (* Manual stop station 3: TRUE = Stop *)
    Sensor1 : BOOL;                   (* Object detection sensor 1: TRUE = Object *)
    Sensor2 : BOOL;                   (* Object detection sensor 2: TRUE = Object *)
    Sensor3 : BOOL;                   (* Object detection sensor 3: TRUE = Object *)
    Sensor4 : BOOL;                   (* Object detection sensor 4: TRUE = Object *)
    Sensor5 : BOOL;                   (* Object detection sensor 5: TRUE = Object *)
    AutoMode : BOOL;                  (* Auto mode selector: TRUE = Active *)
    ManualMode : BOOL;                (* Manual mode selector: TRUE = Active *)
    Reset : BOOL;                     (* Manual reset for stop latch *)
    
    (* Outputs *)
    ConveyorRunning : BOOL;           (* Conveyor motor: TRUE = Running *)
    ConveyorSpeed : REAL := 2.0;      (* Constant speed: 2 m/s (logical representation) *)
    AlarmActive : BOOL;               (* Alarm: TRUE if stop triggered or mode error *)
    
    (* Internal Variables *)
    StopLatched : BOOL;               (* Latch stop condition until reset *)
    LastReset : BOOL;                 (* For rising edge detection on Reset *)
    AllSensorsActive : BOOL;          (* TRUE if all sensors detect objects *)
    ModeValid : BOOL;                 (* TRUE if exactly one mode is selected *)
END_VAR

(* Initialize outputs *)
ConveyorRunning := FALSE;             (* Start with conveyor stopped *)
AlarmActive := FALSE;                 (* No initial alarm *)
StopLatched := FALSE;                 (* No initial stop condition *)

(* Main logic *)
(* Safety override: Manual stops take highest priority *)
IF StationStop1 OR StationStop2 OR StationStop3 THEN
    (* Any stop station activated: Halt conveyor, latch stop, activate alarm *)
    ConveyorRunning := FALSE;         (* Stop conveyor motor *)
    StopLatched := TRUE;              (* Latch stop until reset *)
    AlarmActive := TRUE;              (* Activate alarm for operator awareness *)
ELSE
    (* Check mode validity: Exactly one mode should be active *)
    (* Prevents ambiguous operation if both or neither selected *)
    ModeValid := (AutoMode XOR ManualMode) OR (NOT AutoMode AND NOT ManualMode);
    
    IF NOT ModeValid THEN
        (* Invalid mode selection: Stop conveyor, activate alarm *)
        ConveyorRunning := FALSE;     (* Stop conveyor *)
        AlarmActive := TRUE;          (* Indicate mode error *)
        StopLatched := TRUE;          (* Latch stop until reset *)
    ELSIF StopLatched THEN
        (* Stop latched from previous condition: Keep conveyor off *)
        ConveyorRunning := FALSE;     (* Maintain stopped state *)
        AlarmActive := TRUE;          (* Keep alarm active *)
    ELSE
        (* Safe to evaluate mode-based operation *)
        IF AutoMode THEN
            (* Automatic mode: Run only if all sensors detect objects *)
            AllSensorsActive := Sensor1 AND Sensor2 AND Sensor3 AND Sensor4 AND Sensor5;
            IF AllSensorsActive THEN
                ConveyorRunning := TRUE;  (* Run conveyor *)
                AlarmActive := FALSE;     (* Clear alarm *)
            ELSE
                ConveyorRunning := FALSE; (* Stop conveyor *)
                AlarmActive := FALSE;     (* No alarm unless latched *)
            END_IF;
        ELSIF ManualMode THEN
            (* Manual mode: Run conveyor unless stopped *)
            ConveyorRunning := TRUE;     (* Run conveyor *)
            AlarmActive := FALSE;        (* Clear alarm *)
        ELSE
            (* No mode selected: Stop conveyor as fallback *)
            ConveyorRunning := FALSE;    (* Stop conveyor *)
            AlarmActive := FALSE;        (* No alarm *)
        END_IF;
    END_IF;
END_IF;

(* Manual reset logic *)
(* Clears StopLatched and AlarmActive after operator verification *)
IF Reset AND NOT LastReset AND NOT (StationStop1 OR StationStop2 OR StationStop3) THEN
    (* Rising edge on Reset, no active stops: Clear latched state *)
    StopLatched := FALSE;             (* Allow operation *)
    AlarmActive := FALSE;             (* Reset alarm *)
END_IF;
LastReset := Reset;

(* Ensure safe state on power-up or PLC stop *)
(* Conveyor remains stopped until valid conditions met *)
IF NOT ModeValid OR StopLatched OR (StationStop1 OR StationStop2 OR StationStop3) THEN
    ConveyorRunning := FALSE;         (* Enforce stopped state *)
END_IF;

(* Maintain constant speed *)
(* ConveyorSpeed is a logical representation, set to 2.0 m/s *)
(* Actual speed control handled by motor drive configuration *)
ConveyorSpeed := 2.0;

END_PROGRAM
