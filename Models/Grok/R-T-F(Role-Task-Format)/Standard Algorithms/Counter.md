(* IEC 61131-3 Structured Text function block for a digital counter *)
(* Supports up/down counting with configurable limits and step size *)
(* Optimized for reliable execution in scan-cycle-driven PLC environments *)

FUNCTION_BLOCK DigitalCounter
VAR_INPUT
    Enable : BOOL; (* TRUE to enable counting *)
    CountUp : BOOL; (* TRUE for up counting, FALSE for down counting *)
    Reset : BOOL; (* TRUE to reset counter to InitialValue *)
    InitialValue : DINT := 0; (* Initial count value *)
    StepSize : DINT := 1; (* Increment/decrement step size *)
    MaxValue : DINT := 1000; (* Maximum allowed count *)
    MinValue : DINT := -1000; (* Minimum allowed count *)
END_VAR

VAR_OUTPUT
    CurrentValue : DINT; (* Current count value *)
    MaxReached : BOOL; (* TRUE if CurrentValue >= MaxValue *)
    MinReached : BOOL; (* TRUE if CurrentValue <= MinValue *)
    Fault : BOOL; (* TRUE if invalid inputs detected *)
END_VAR

VAR
    LastEnable : BOOL; (* Tracks previous Enable state for edge detection *)
    Initialized : BOOL := FALSE; (* Tracks if counter is initialized *)
END_VAR

(* Input validation and initialization *)
IF Enable AND NOT LastEnable THEN
    (* Validate inputs on rising edge of Enable *)
    IF MinValue > MaxValue OR StepSize <= 0 OR InitialValue < MinValue OR InitialValue > MaxValue THEN
        Fault := TRUE;
        CurrentValue := InitialValue;
        MaxReached := FALSE;
        MinReached := FALSE;
        Initialized := FALSE;
        RETURN;
    END_IF;
    
    (* Initialize counter *)
    CurrentValue := InitialValue;
    MaxReached := (CurrentValue >= MaxValue);
    MinReached := (CurrentValue <= MinValue);
    Fault := FALSE;
    Initialized := TRUE;
END_IF;

(* Store Enable state for edge detection *)
LastEnable := Enable;

(* Reset handling *)
IF Reset THEN
    IF MinValue <= InitialValue AND InitialValue <= MaxValue THEN
        CurrentValue := InitialValue;
        MaxReached := (CurrentValue >= MaxValue);
        MinReached := (CurrentValue <= MinValue);
        Fault := FALSE;
        Initialized := TRUE;
    ELSE
        Fault := TRUE;
        Initialized := FALSE;
    END_IF;
END_IF;

(* Counting logic *)
IF Enable AND Initialized AND NOT Fault THEN
    IF CountUp THEN
        (* Up counting *)
        IF CurrentValue + StepSize <= MaxValue THEN
            CurrentValue := CurrentValue + StepSize;
        ELSE
            CurrentValue := MaxValue; (* Clamp at MaxValue *)
            MaxReached := TRUE;
        END_IF;
    ELSE
        (* Down counting *)
        IF CurrentValue - StepSize >= MinValue THEN
            CurrentValue := CurrentValue - StepSize;
        ELSE
            CurrentValue := MinValue; (* Clamp at MinValue *)
            MinReached := TRUE;
        END_IF;
    END_IF;
    
    (* Update status flags *)
    MaxReached := (CurrentValue >= MaxValue);
    MinReached := (CurrentValue <= MinValue);
END_IF;

(* Clear outputs when disabled *)
IF NOT Enable THEN
    MaxReached := FALSE;
    MinReached := FALSE;
    (* CurrentValue and Fault retain their last values *)
END_IF;

(* Performance Notes *)
(* - Executes in a single PLC scan cycle, ensuring deterministic behavior. *)
(* - Uses DINT for wide range (-2^31 to 2^31-1), suitable for most counting tasks. *)
(* - Edge detection on Enable prevents repeated initialization during continuous operation. *)
(* - Fault handling ensures robustness against invalid configurations. *)
(* - Modular design allows reuse in various automation tasks (e.g., production tracking, cycle counting). *)
END_FUNCTION_BLOCK
