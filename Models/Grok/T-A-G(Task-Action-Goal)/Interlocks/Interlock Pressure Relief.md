(* Program: Pressurized Vessel Overpressure Protection Interlock *)
PROGRAM PRG_VesselPressureRelief
VAR
    (* Inputs *)
    Execute : BOOL;                    (* Enable interlock logic *)
    PT_101 : REAL;                     (* Pressure from PT sensor, bar *)
    ValveStatus : BOOL;                (* Relief valve feedback: TRUE = Open, FALSE = Closed *)
    Reset : BOOL;                      (* Manual reset input *)
    
    (* Outputs *)
    RELIEF_VALVE : BOOL;               (* Relief valve command: TRUE = Open, FALSE = Closed *)
    Shutdown : BOOL;                   (* Shutdown state: TRUE = Active *)
    Alarm : BOOL;                      (* Safety alarm *)
    Error : BOOL;                      (* Error flag *)
    ErrorID : DWORD;                   (* Error code *)
    
    (* Internal Variables *)
    LastReset : BOOL;                  (* For rising edge detection on Reset *)
    ValveLatched : BOOL;               (* Latch valve open condition *)
    
    (* Constants *)
    HIGH_PRESSURE_LIMIT : REAL := 15.0; (* High pressure limit: 15 bar *)
    RESET_PRESSURE_THRESHOLD : REAL := 12.0; (* Reset pressure threshold: 12 bar *)
    PRESSURE_SENSOR_MIN : REAL := 0.0; (* Minimum valid sensor reading *)
    PRESSURE_SENSOR_MAX : REAL := 50.0; (* Maximum valid sensor reading *)
END_VAR

(* Initialize outputs *)
RELIEF_VALVE := FALSE; (* Default: Valve closed *)
Shutdown := FALSE;
Alarm := FALSE;
Error := FALSE;
ErrorID := 0;
ValveLatched := FALSE;

(* Main interlock logic *)
IF Execute THEN
    (* Validate sensor input *)
    IF PT_101 < PRESSURE_SENSOR_MIN OR PT_101 > PRESSURE_SENSOR_MAX THEN
        Error := TRUE;
        ErrorID := 16#80010000; (* Invalid sensor reading *)
        RELIEF_VALVE := TRUE;
        Shutdown := TRUE;
        ValveLatched := TRUE;
        Alarm := TRUE;
        RETURN;
    END_IF;

    (* Validate valve status *)
    IF ValveStatus <> RELIEF_VALVE THEN
        Error := TRUE;
        ErrorID := 16#80020000; (* Valve malfunction *)
        RELIEF_VALVE := TRUE;
        Shutdown := TRUE;
        ValveLatched := TRUE;
        Alarm := TRUE;
        RETURN;
    END_IF;

    (* Interlock: High Pressure (>15 bar) *)
    IF PT_101 > HIGH_PRESSURE_LIMIT THEN
        RELIEF_VALVE := TRUE;
        Shutdown := TRUE;
        ValveLatched := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Maintain latched valve open state *)
    IF ValveLatched THEN
        RELIEF_VALVE := TRUE;
        Shutdown := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Manual reset logic *)
    IF Reset AND NOT LastReset AND NOT Error THEN
        (* Rising edge on Reset *)
        IF PT_101 <= RESET_PRESSURE_THRESHOLD THEN
            (* Allow reset only when pressure is safe *)
            ValveLatched := FALSE;
            Shutdown := FALSE;
            RELIEF_VALVE := FALSE; (* Close valve after reset *)
            Alarm := FALSE;
        ELSE
            Error := TRUE;
            ErrorID := 16#80030000; (* Reset attempted with unsafe pressure *)
        END_IF;
    END_IF;
    LastReset := Reset;

ELSE
    (* Interlocks disabled: Safe state *)
    RELIEF_VALVE := TRUE; (* Fail-safe: Valve open *)
    Shutdown := TRUE;
    ValveLatched := TRUE;
    Alarm := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_PROGRAM
