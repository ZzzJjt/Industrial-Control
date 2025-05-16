(* IEC 61131-3 Structured Text: Flexible Timer Function Block *)
(* Purpose: Provides a configurable timer for sequencing, delays, and safety checks *)

FUNCTION_BLOCK FlexibleTimer
VAR_INPUT
    Start : BOOL;                   (* TRUE to start or continue timing *)
    Stop : BOOL;                    (* TRUE to pause or reset timer *)
    PresetTime : TIME;              (* Configurable timer duration, e.g., T#5s *)
END_VAR
VAR_OUTPUT
    Running : BOOL;                 (* TRUE while timer is counting *)
    Done : BOOL;                    (* TRUE when elapsed time >= PresetTime *)
    RemainingTime : TIME;           (* Remaining time until Done *)
    Error : BOOL;                   (* TRUE if PresetTime is invalid *)
END_VAR
VAR
    StartTime : TIME;               (* Time when timer last started *)
    ElapsedTime : TIME;             (* Accumulated elapsed time *)
    LastScanTime : TIME;            (* Time of last scan cycle *)
    TimerActive : BOOL := FALSE;    (* Internal flag for timer state *)
    Initialized : BOOL := FALSE;    (* TRUE after first run *)
END_VAR

(* Main Logic *)
IF NOT Initialized THEN
    (* Initialize on first run *)
    LastScanTime := TIME();  (* Get current PLC time *)
    Initialized := TRUE;
END_IF;

(* Input Validation *)
IF PresetTime < T#0s THEN
    (* Invalid PresetTime *)
    Error := TRUE;
    Running := FALSE;
    Done := FALSE;
    RemainingTime := T#0s;
    ElapsedTime := T#0s;
    TimerActive := FALSE;
ELSE
    Error := FALSE;
    
    (* Update Current Time *)
    CurrentTime := TIME();  (* Get current PLC time *)
    DeltaT := CurrentTime - LastScanTime;  (* Time since last scan *)
    LastScanTime := CurrentTime;
    
    (* Timer Control Logic *)
    IF Stop THEN
        IF NOT Start THEN
            (* Reset timer when Stop = TRUE and Start = FALSE *)
            ElapsedTime := T#0s;
            TimerActive := FALSE;
            Running := FALSE;
            Done := FALSE;
        ELSE
            (* Pause timer when Stop = TRUE and Start = TRUE *)
            TimerActive := FALSE;
            Running := FALSE;
        END_IF;
    ELSIF Start AND NOT TimerActive THEN
        (* Start or resume timer *)
        TimerActive := TRUE;
        Running := TRUE;
        IF ElapsedTime = T#0s THEN
            StartTime := CurrentTime;
        END_IF;
    END_IF;
    
    (* Update Elapsed Time *)
    IF TimerActive AND DeltaT > T#0s THEN
        ElapsedTime := ElapsedTime + DeltaT;
    END_IF;
    
    (* Check Timeout *)
    IF ElapsedTime >= PresetTime THEN
        Done := TRUE;
        Running := FALSE;
        TimerActive := FALSE;
        ElapsedTime := PresetTime;  (* Clamp to PresetTime *)
    ELSE
        Done := FALSE;
    END_IF;
    
    (* Calculate Remaining Time *)
    IF ElapsedTime < PresetTime THEN
        RemainingTime := PresetTime - ElapsedTime;
    ELSE
        RemainingTime := T#0s;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Flexible timer for industrial automation, supporting start/stop control.
   - Inputs:
     - Start: TRUE to start or continue timing.
     - Stop: TRUE to pause (with Start = TRUE) or reset (with Start = FALSE).
     - PresetTime: TIME duration (e.g., T#5s).
   - Outputs:
     - Running: TRUE while timer is counting.
     - Done: TRUE when ElapsedTime >= PresetTime.
     - RemainingTime: TIME left until Done.
     - Error: TRUE if PresetTime < T#0s.
   - Logic:
     - Starts timing when Start = TRUE and Stop = FALSE.
     - Pauses when Stop = TRUE, retains ElapsedTime.
     - Resets when Stop = TRUE and Start = FALSE.
     - Tracks ElapsedTime using PLC clock (TIME()).
   - Safety:
     - Validates PresetTime to prevent invalid durations.
     - Resets outputs on error or disable for safe state.
     - Clamps ElapsedTime to PresetTime to avoid overflow.
   - Usage:
     - Ideal for sequencing operations, delays, or safety checks in PLC systems.
     - Reusable in scan-cycle-driven environments (e.g., CODESYS, TwinCAT).
   - Maintenance:
     - Modular design with clear comments aids debugging.
     - Running, Done, and Error flags support diagnostics via HMI/logging.
   - Platform Notes:
     - Uses TIME for durations; assumes TIME() function for PLC clock.
     - Single-cycle updates ensure scan-cycle safety.
     - Max PresetTime depends on PLC TIME resolution (typically 2^31 ms).
*)
END_FUNCTION_BLOCK
