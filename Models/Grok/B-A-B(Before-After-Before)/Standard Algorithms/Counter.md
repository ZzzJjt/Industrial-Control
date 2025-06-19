(* IEC 61131-3 Structured Text: Flexible Digital Counter Function Block *)
(* Purpose: Provides configurable up/down counting with overflow/underflow protection *)

FUNCTION_BLOCK DigitalCounter
VAR_INPUT
    Enable : BOOL;                  (* TRUE to enable counting *)
    CountUp : BOOL;                 (* TRUE for up, FALSE for down *)
    StepSize : INT := 1;            (* Step size for counting, must be positive *)
    InitValue : INT;                (* Initial counter value *)
    Reset : BOOL;                   (* TRUE to reset counter to InitValue *)
    MaxValue : INT := 32767;        (* Maximum allowed counter value *)
    MinValue : INT := -32768;       (* Minimum allowed counter value *)
END_VAR
VAR_OUTPUT
    CurrentValue : INT;             (* Current counter value *)
    AtMax : BOOL;                   (* TRUE if CurrentValue = MaxValue *)
    AtMin : BOOL;                   (* TRUE if CurrentValue = MinValue *)
    Error : BOOL;                   (* TRUE if input validation fails *)
END_VAR
VAR
    Initialized : BOOL := FALSE;    (* Tracks if counter is initialized *)
END_VAR

(* Main Logic *)
IF Reset THEN
    (* Reset counter to InitValue *)
    Initialized := FALSE;
END_IF;

IF NOT Enable THEN
    (* Disable counting, maintain outputs *)
    AtMax := CurrentValue >= MaxValue;
    AtMin := CurrentValue <= MinValue;
    Error := FALSE;
ELSE
    (* Input Validation *)
    IF StepSize <= 0 OR MaxValue < MinValue OR InitValue < MinValue OR InitValue > MaxValue THEN
        (* Invalid inputs: negative StepSize, invalid bounds, or InitValue out of range *)
        Error := TRUE;
        AtMax := FALSE;
        AtMin := FALSE;
        CurrentValue := 0;
    ELSE
        Error := FALSE;
        
        (* Initialize counter on first enable after reset *)
        IF NOT Initialized THEN
            CurrentValue := InitValue;
            Initialized := TRUE;
        END_IF;
        
        (* Counting Logic *)
        IF CountUp THEN
            (* Up counting *)
            IF CurrentValue <= MaxValue - StepSize THEN
                CurrentValue := CurrentValue + StepSize;
            ELSE
                CurrentValue := MaxValue;  (* Clamp to MaxValue *)
            END_IF;
        ELSE
            (* Down counting *)
            IF CurrentValue >= MinValue + StepSize THEN
                CurrentValue := CurrentValue - StepSize;
            ELSE
                CurrentValue := MinValue;  (* Clamp to MinValue *)
            END_IF;
        END_IF;
        
        (* Update Status Flags *)
        AtMax := CurrentValue >= MaxValue;
        AtMin := CurrentValue <= MinValue;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Flexible digital counter for industrial automation, supporting up/down counting.
   - Inputs:
     - Enable: Enables counting when TRUE.
     - CountUp: TRUE for up counting, FALSE for down counting.
     - StepSize: Positive integer step size (default 1).
     - InitValue: Initial counter value.
     - Reset: Resets counter to InitValue when TRUE.
     - MaxValue: Upper bound (default 32767).
     - MinValue: Lower bound (default -32768).
   - Outputs:
     - CurrentValue: Current counter value.
     - AtMax: TRUE when CurrentValue reaches MaxValue.
     - AtMin: TRUE when CurrentValue reaches MinValue.
     - Error: TRUE if inputs are invalid (StepSize <= 0, MaxValue < MinValue, InitValue out of bounds).
   - Features:
     - Configurable counting direction (up/down).
     - Customizable step size and initial value.
     - Overflow/underflow protection by clamping to MaxValue/MinValue.
   - Safety:
     - Validates inputs to prevent invalid operations.
     - Clamps CurrentValue to avoid overflow/underflow.
     - Resets properly with InitValue on Reset.
   - Usage:
     - Suitable for tracking events, cycles, or quantities in PLC systems.
     - Reusable in scan-cycle-driven environments (e.g., CODESYS, TwinCAT).
   - Maintenance:
     - Modular design with clear comments aids debugging.
     - Error and status flags (AtMax, AtMin) support diagnostics.
*)
END_FUNCTION_BLOCK
