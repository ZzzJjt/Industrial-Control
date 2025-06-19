(* Program: Subsea Gas Wellhead Emergency Interlock System *)
PROGRAM PRG_SubseaWellheadInterlocks
VAR
    (* Inputs *)
    Execute : BOOL;                    (* Enable interlock logic *)
    PT_101 : REAL;                     (* Pressure from PT sensor, psi *)
    TT_101 : REAL;                     (* Temperature from TT sensor, °C *)
    FT_101 : REAL;                     (* Flow rate from FT sensor, m³/h *)
    Reset : BOOL;                      (* Manual reset input *)
    
    (* Outputs *)
    MV_101 : BOOL;                     (* Master Valve: TRUE = Open, FALSE = Closed *)
    Shutdown : BOOL;                   (* Shutdown state: TRUE = Active *)
    Alarm : BOOL;                      (* Safety alarm *)
    Error : BOOL;                      (* Error flag *)
    ErrorID : DWORD;                   (* Error code *)
    
    (* Internal Variables *)
    LastReset : BOOL;                  (* For rising edge detection on Reset *)
    ShutdownLatched : BOOL;            (* Latch shutdown condition *)
    
    (* Constants *)
    MAX_PRESSURE : REAL := 1500.0;     (* Max pressure limit: 1500 psi *)
    MIN_FLOW : REAL := 10.0;           (* Min safe flow rate: 10 m³/h *)
    MAX_TEMPERATURE : REAL := 120.0;   (* Max temperature limit: 120°C *)
END_VAR

(* Initialize outputs *)
MV_101 := TRUE; (* Default: Master valve open *)
Shutdown := FALSE;
Alarm := FALSE;
Error := FALSE;
ErrorID := 0;
ShutdownLatched := FALSE;

(* Main interlock logic *)
IF Execute THEN
    (* Validate sensor inputs *)
    IF PT_101 < 0.0 OR PT_101 > 2000.0 OR 
       TT_101 < -50.0 OR TT_101 > 200.0 OR 
       FT_101 < 0.0 OR FT_101 > 1000.0 THEN
        Error := TRUE;
        ErrorID := 16#80010000; (* Invalid sensor reading *)
        MV_101 := FALSE;
        Shutdown := TRUE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
        RETURN;
    END_IF;

    (* Interlock 1: High Pressure (>1500 psi) *)
    IF PT_101 > MAX_PRESSURE THEN
        MV_101 := FALSE;
        Shutdown := TRUE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Interlock 2: Low Flow Rate (<10 m³/h) *)
    IF FT_101 < MIN_FLOW THEN
        MV_101 := FALSE;
        Shutdown := TRUE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Interlock 3: High Temperature (>120°C) *)
    IF TT_101 > MAX_TEMPERATURE THEN
        MV_101 := FALSE;
        Shutdown := TRUE;
        ShutdownLatched := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Maintain latched shutdown *)
    IF ShutdownLatched THEN
        MV_101 := FALSE;
        Shutdown := TRUE;
        Alarm := TRUE;
    END_IF;

    (* Manual reset logic *)
    IF Reset AND NOT LastReset AND NOT Error THEN
        (* Rising edge on Reset *)
        IF ShutdownLatched THEN
            (* Allow reset only after operator review *)
            ShutdownLatched := FALSE;
            Shutdown := FALSE;
            MV_101 := TRUE; (* Reopen valve after reset *)
            Alarm := FALSE;
        END_IF;
    END_IF;
    LastReset := Reset;

ELSE
    (* Interlocks disabled: Safe state *)
    MV_101 := FALSE;
    Shutdown := TRUE;
    ShutdownLatched := TRUE;
    Alarm := FALSE;
    Error := FALSE;
    ErrorID := 0;
END_IF;

END_PROGRAM
