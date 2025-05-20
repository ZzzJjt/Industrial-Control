(* IEC 61131-3 Structured Text function block for a customizable on-delay timer *)
(* Tracks elapsed time with configurable duration, start/stop control, and status flags *)
(* Optimized for scan-cycle compatibility and industrial control applications *)

FUNCTION_BLOCK CustomTimer
VAR_INPUT
    Start : BOOL; (* TRUE to start or continue timing *)
    Stop : BOOL; (* TRUE to stop or reset timer *)
    PresetTime : TIME; (* Desired timer duration *)
END_VAR

VAR_OUTPUT
    Running : BOOL; (* TRUE while timer is active *)
    Done : BOOL; (* TRUE when elapsed time >= PresetTime *)
    RemainingTime : TIME; (* Time remaining until PresetTime *)
    Fault : BOOL := FALSE; (* TRUE if invalid PresetTime *)
END_VAR

VAR
    StartTime : TIME; (* System time when timer started *)
    ElapsedTime : TIME; (* Current elapsed time *)
    LastStart : BOOL; (* Tracks previous Start state for edge detection *)
    Initialized : BOOL := FALSE; (* Tracks if timer is initialized *)
    CurrentTime : TIME; (* Current system time *)
END_VAR

(* Get current system time *)
(* Note: SYSTEM_TIME() is a placeholder; replace with PLC-specific function *)
CurrentTime := SYSTEM_TIME();

(* Input validation *)
IF PresetTime < T#0s THEN
    Fault := TRUE;
    Running := FALSE;
    Done := FALSE;
    RemainingTime := T#0s;
    Initialized := FALSE;
    RETURN;
END_IF;

(* Stop or reset handling *)
IF Stop THEN
    Running := FALSE;
    Done := FALSE;
    RemainingTime := PresetTime;
    ElapsedTime := T#0s;
    Initialized := FALSE;
    Fault := FALSE;
    LastStart := FALSE;
    RETURN;
END_IF;

(* Start edge detection and timer initialization *)
IF Start AND NOT LastStart AND NOT Initialized THEN
    (* Rising edge of Start *)
    StartTime := CurrentTime;
    ElapsedTime := T#0s;
    Running := TRUE;
    Done := FALSE;
    RemainingTime := PresetTime;
    Initialized := TRUE;
    Fault := FALSE;
END_IF;

(* Update last Start state *)
LastStart := Start;

(* Timer logic *)
IF Running AND Initialized THEN
    (* Calculate elapsed time *)
    ElapsedTime := CurrentTime - StartTime;
    
    (* Check if timer has reached PresetTime *)
    IF ElapsedTime >= PresetTime THEN
        Running := FALSE;
        Done := TRUE;
        ElapsedTime := PresetTime;
        RemainingTime := T#0s;
        Initialized := FALSE;
    ELSE
        RemainingTime := PresetTime - ElapsedTime;
    END_IF;
    
    (* Stop timing if Start is FALSE *)
    IF NOT Start THEN
        Running := FALSE;
        Done := FALSE;
        Initialized := FALSE;
    END_IF;
END_IF;

(* Clear outputs when not running or done *)
IF NOT Running AND NOT Done THEN
    RemainingTime := PresetTime;
    ElapsedTime := T#0s;
END_IF;

(* Execution Notes *)
(* - Executes in a single scan cycle (<1 ms), suitable for 10-50 ms PLC scan times. *)
(* - Uses SYSTEM_TIME() for accurate time tracking; replace with PLC-specific function. *)
(* - Validates PresetTime (>= 0) to prevent invalid timing behavior. *)
(* - Edge detection on Start ensures clean initialization, avoiding retrigger issues. *)
(* - Modular design supports reuse in applications like sequencing, delays, or alarms. *)
END_FUNCTION_BLOCK
