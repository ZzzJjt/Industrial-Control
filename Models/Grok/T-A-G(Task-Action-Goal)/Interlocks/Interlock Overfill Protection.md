(* Program: Process Vessel Overfill Protection Interlock *)
PROGRAM PRG_VesselOverfillProtection
VAR
    (* Inputs *)
    Execute : BOOL;                    (* Enable interlock logic *)
    LevelSensor : REAL;                (* Vessel liquid level, % (0-100) *)
    ValveStatus : BOOL;                (* Inlet valve feedback: TRUE = Open, FALSE = Closed *)
    Reset : BOOL;                      (* Manual reset input *)
    
    (* Outputs *)
    INLET_VALVE : BOOL;                (* Inlet valve command: TRUE = Open, FALSE = Closed *)
    Shutdown : BOOL;                   (* Shutdown state: TRUE = Active *)
    Alarm : BOOL;                      (* Safety alarm *)
    Error : BOOL;                      (* Error flag *)
    ErrorID : DWORD;                   (* Error code *)
    
    (* Internal Variables *)
    LastReset : BOOL;                  (* For rising edge detection on Reset *)
    ShutdownLatched : BOOL;            (* Latch shutdown condition *)
    
    (* Constants *)
    HIGH_LEVEL_SETPOINT : REAL := 90.0; (* High level setpoint: 90% *)
    RESET_LEVEL_THRESHOLD : REAL := 80.0; (* Reset level threshold: 80% *)
    LEVEL_SENSOR_MIN : REAL := 0.0;    (* Minimum valid sensor reading *)
    LEVEL_SENSOR_MAX : REAL := 100.0;  (* Maximum valid sensor reading *)
END_VAR

(* Initialize outputs *)
INLET_VALVE := TRUE; (* Default: Valve open *)
Shutdown := FALSE;
Alarm := FALSE;
Error := FALSE;
ErrorID := 0;
ShutdownLatched := FALSE;

(* Main interlock logic *)
IF Execute THEN
    (* Validate sensor input *)
    IF LevelSensor < LEVEL_SENSOR_MIN OR LevelSensor > LEVEL_SENSOR_MAX THEN
        Error := TRUE;
        ErrorID := 16#80010000; (* Invalid sensor reading *)
        INLET_VALVE := FALSE;
        Shutdown := TRUE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
        RETURN;
    END_IF;

    (* Validate valve status *)
    IF ValveStatus <> INLET_VALVE THEN
        Error := TRUE;
        ErrorID := 16#80020000; (* Valve malfunction *)
        INLET_VALVE := FALSE;
        Shutdown := TRUE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
        RETURN;
    END_IF;

    (* Interlock: High Level (>90%) *)
    IF LevelSensor > HIGH_LEVEL_SETPOINT THEN
        INLET_VALVE := FALSE;
        Shutdown := TRUE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Maintain latched shutdown *)
    IF ShutdownLatched THEN
        INLET_VALVE := FALSE;
        Shutdown := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Manual reset logic *)
    IF Reset AND NOT LastReset AND NOT Error THEN
        (* Rising edge on Reset *)
        IF LevelSensor <= RESET_LEVEL_THRESHOLD THEN
            (* Allow reset only when level is safe *)
            ShutdownLatched := FALSE;
            Shutdown := FALSE;
            INLET_VALVE := TRUE; (* Reopen valve after reset *)
            Alarm := FALSE;
        ELSE
            Error := TRUE;
            ErrorID := 16#80030000; (* Reset attempted with unsafe level *)
        END_IF;
    END_IF;
    LastReset := Reset;

ELSE
    (* Interlocks disabled: Safe state *)
    INLET_VALVE := FALSE;
    Shutdown := TRUE;
    ShutdownLatched := TRUE;
    Alarm := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_PROGRAM
