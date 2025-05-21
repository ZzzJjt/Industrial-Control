(* Program: Chemical Reactor Safety Door Interlock System *)
PROGRAM PRG_ReactorSafetyDoorInterlocks
VAR
    (* Inputs *)
    Execute : BOOL;                    (* Enable interlock logic *)
    DOOR_1_CLOSED : BOOL;             (* Door 1 closed sensor: TRUE = Closed *)
    DOOR_2_CLOSED : BOOL;             (* Door 2 closed sensor: TRUE = Closed *)
    DOOR_3_CLOSED : BOOL;             (* Door 3 closed sensor: TRUE = Closed *)
    Reset : BOOL;                      (* Manual reset input *)
    
    (* Outputs *)
    ALLOW_START : BOOL;                (* Allow reactor startup: TRUE = Permitted *)
    ReactorRunning : BOOL;             (* Reactor state: TRUE = Running *)
    EMERGENCY_SHUTDOWN : BOOL;         (* Emergency shutdown state: TRUE = Active *)
    Heater : BOOL;                     (* Heater state: TRUE = On *)
    Agitator : BOOL;                   (* Agitator state: TRUE = On *)
    Alarm : BOOL;                      (* Safety alarm *)
    Error : BOOL;                      (* Error flag *)
    ErrorID : DWORD;                   (* Error code *)
    
    (* Internal Variables *)
    LastReset : BOOL;                  (* For rising edge detection on Reset *)
    ShutdownLatched : BOOL;            (* Latch shutdown condition *)
    DoorsAllClosed : BOOL;             (* Combined door status *)
END_VAR

(* Initialize outputs *)
ALLOW_START := FALSE;
ReactorRunning := FALSE;
EMERGENCY_SHUTDOWN := FALSE;
Heater := FALSE;
Agitator := FALSE;
Alarm := FALSE;
Error := FALSE;
ErrorID := 0;
ShutdownLatched := FALSE;

(* Main interlock logic *)
IF Execute THEN
    (* Validate sensor inputs *)
    (* Assume loss of signal (FALSE) indicates open or faulty sensor *)
    IF NOT (DOOR_1_CLOSED OR NOT DOOR_1_CLOSED) OR
       NOT (DOOR_2_CLOSED OR NOT DOOR_2_CLOSED) OR
       NOT (DOOR_3_CLOSED OR NOT DOOR_3_CLOSED) THEN
        Error := TRUE;
        ErrorID := 16#80010000; (* Invalid sensor input *)
        ALLOW_START := FALSE;
        ReactorRunning := FALSE;
        EMERGENCY_SHUTDOWN := TRUE;
        Heater := FALSE;
        Agitator := FALSE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
        RETURN;
    END_IF;

    (* Check if all doors are closed *)
    DoorsAllClosed := DOOR_1_CLOSED AND DOOR_2_CLOSED AND DOOR_3_CLOSED;

    (* Interlock 1: Prevent startup if any door is open *)
    ALLOW_START := DoorsAllClosed AND NOT ShutdownLatched;

    (* Interlock 2: Shutdown if any door opens during operation *)
    IF ReactorRunning AND NOT DoorsAllClosed THEN
        ReactorRunning := FALSE;
        EMERGENCY_SHUTDOWN := TRUE;
        Heater := FALSE;
        Agitator := FALSE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Maintain latched shutdown *)
    IF ShutdownLatched THEN
        ReactorRunning := FALSE;
        EMERGENCY_SHUTDOWN := TRUE;
        Heater := FALSE;
        Agitator := FALSE;
        ALLOW_START := FALSE;
        Alarm := TRUE;
    END_IF;

    (* Manual reset logic *)
    IF Reset AND NOT LastReset AND NOT Error THEN
        (* Rising edge on Reset *)
        IF DoorsAllClosed THEN
            (* Allow reset only when all doors are closed *)
            ShutdownLatched := FALSE;
            EMERGENCY_SHUTDOWN := FALSE;
            Alarm := FALSE;
        ELSE
            Error := TRUE;
            ErrorID := 16#80020000; (* Reset attempted with open door *)
        END_IF;
    END_IF;
    LastReset := Reset;

ELSE
    (* Interlocks disabled: Safe state *)
    ALLOW_START := FALSE;
    ReactorRunning := FALSE;
    EMERGENCY_SHUTDOWN := TRUE;
    Heater := FALSE;
    Agitator := FALSE;
    ShutdownLatched := TRUE;
    Alarm := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_PROGRAM
