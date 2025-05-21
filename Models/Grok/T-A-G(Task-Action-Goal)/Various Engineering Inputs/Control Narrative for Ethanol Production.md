(* IEC 61131-3 Structured Text Function Block for Timer *)
(* Implements an edge-triggered timer for delay control, sequencing, or timeout *)
(* Uses system clock for precise timing, safe for PLC scan cycles *)
(* Conforms to industrial automation best practices *)

FUNCTION_BLOCK Timer
VAR_INPUT
    Start: BOOL;               (* TRUE to start timer on rising edge *)
    Stop: BOOL;                (* TRUE to halt and reset timer *)
    PresetTime: TIME;          (* Duration to time, e.g., T#5s *)
END_VAR

VAR_OUTPUT
    Running: BOOL;             (* TRUE when timer is active *)
    Done: BOOL;                (* TRUE when PresetTime is reached *)
    RemainingTime: TIME;       (* Time left before timeout *)
END_VAR

VAR
    PrevStart: BOOL;           (* Previous state of Start for edge detection *)
    StartTime: TIME;           (* System time when timer started *)
    ElapsedTime: TIME;         (* Time elapsed since start *)
    ValidInput: BOOL;          (* TRUE if inputs are valid *)
    Initialized: BOOL;         (* TRUE after first valid cycle *)
END_VAR

(* Step 1: Input Validation *)
(* Ensure PresetTime is non-negative *)
ValidInput := (PresetTime >= T#0s);

(* Step 2: Handle Stop *)
(* Halt and reset timer when Stop is TRUE *)
IF Stop THEN
    Running := FALSE;
    Done := FALSE;
    ElapsedTime := T#0s;
    RemainingTime := PresetTime;
    PrevStart := FALSE;
    Initialized := FALSE;
    RETURN;
END_IF;

(* Step 3: Check Valid Input *)
(* Exit if inputs are invalid *)
IF NOT ValidInput THEN
    Running := FALSE;
    Done := FALSE;
    ElapsedTime := T#0s;
    RemainingTime := T#0s;
    PrevStart := FALSE;
    Initialized := FALSE;
    RETURN;
END_IF;

(* Step 4: Detect Rising Edge of Start *)
(* Initialize timer on rising edge of Start *)
IF Start AND NOT PrevStart AND NOT Running THEN
    StartTime := GET_TIME();   (* Capture current system time *)
    ElapsedTime := T#0s;
    Running := TRUE;
    Done := FALSE;
    Initialized := TRUE;
END_IF;

(* Step 5: Update Timer *)
(* Compute elapsed and remaining time while running *)
IF Running THEN
    IF Initialized THEN
        (* Calculate elapsed time using system clock *)
        ElapsedTime := GET_TIME() - StartTime;
        
        (* Check if timer has reached PresetTime *)
        IF ElapsedTime >= PresetTime THEN
            ElapsedTime := PresetTime;  (* Clamp to PresetTime *)
            Running := FALSE;
            Done := TRUE;
        END_IF;
        
        (* Compute remaining time *)
        IF PresetTime > ElapsedTime THEN
            RemainingTime := PresetTime - ElapsedTime;
        ELSE
            RemainingTime := T#0s;
        END_IF;
    ELSE
        (* Handle case where GET_TIME failed to initialize *)
        Running := FALSE;
        Done := FALSE;
        ElapsedTime := T#0s;
        RemainingTime := PresetTime;
    END_IF;
END_IF;

(* Step 6: Update Previous Start State *)
(* Track Start for edge detection *)
PrevStart := Start;

(* Step 7: Safety Check *)
(* Ensure outputs are consistent *)
IF Running AND Done THEN
    Running := FALSE;  (* Done implies timer is no longer running *)
END_IF;

IF ElapsedTime < T#0s OR RemainingTime < T#0s THEN
    Running := FALSE;
    Done := FALSE;
    ElapsedTime := T#0s;
    RemainingTime := PresetTime;
END_IF;

END_FUNCTION_BLOCK
