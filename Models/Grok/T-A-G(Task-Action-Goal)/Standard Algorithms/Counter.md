(* IEC 61131-3 Structured Text Function Block for Configurable Digital Counter *)
(* Supports up/down counting with bounds, step size, and reset functionality *)
(* Conforms to industrial automation best practices for PLC applications *)

FUNCTION_BLOCK DigitalCounter
VAR_INPUT
    Enable: BOOL;          (* TRUE to activate counter *)
    CountUp: BOOL;         (* TRUE for up counting, FALSE for down counting *)
    StepSize: INT;         (* Increment/decrement amount, must be positive *)
    InitValue: INT;        (* Initial value when reset *)
    Reset: BOOL;           (* TRUE to reset counter to InitValue *)
    MaxValue: INT;         (* Upper bound for counter *)
    MinValue: INT;         (* Lower bound for counter *)
END_VAR

VAR_OUTPUT
    CurrentValue: INT;     (* Current counter value *)
    AtMax: BOOL;           (* TRUE when CurrentValue reaches MaxValue *)
    AtMin: BOOL;           (* TRUE when CurrentValue reaches MinValue *)
END_VAR

VAR
    ValidInput: BOOL;      (* Flag for input validation *)
    PreviousEnable: BOOL;   (* Tracks Enable state for edge detection *)
    TempValue: INT;        (* Temporary value for calculations *)
END_VAR

(* Step 1: Input Validation *)
(* Ensure StepSize is positive and bounds are valid *)
ValidInput := (StepSize > 0) AND (MinValue <= MaxValue) AND (InitValue >= MinValue) AND (InitValue <= MaxValue);

(* Step 2: Handle Reset *)
(* Reset counter to InitValue when Reset is TRUE *)
IF Reset THEN
    CurrentValue := InitValue;
    AtMax := (CurrentValue = MaxValue);
    AtMin := (CurrentValue = MinValue);
    PreviousEnable := FALSE; (* Reset edge detection *)
    RETURN;
END_IF;

(* Step 3: Check Enable and Valid Input *)
(* Exit if not enabled or inputs are invalid *)
IF NOT Enable OR NOT ValidInput THEN
    AtMax := (CurrentValue = MaxValue);
    AtMin := (CurrentValue = MinValue);
    PreviousEnable := Enable;
    RETURN;
END_IF;

(* Step 4: Detect Positive Edge of Enable *)
(* Initialize counter on first enable cycle *)
IF Enable AND NOT PreviousEnable THEN
    CurrentValue := InitValue;
    AtMax := (CurrentValue = MaxValue);
    AtMin := (CurrentValue = MinValue);
    PreviousEnable := TRUE;
    RETURN;
END_IF;

(* Step 5: Counting Logic *)
(* Increment or decrement based on CountUp, apply bounds *)
IF CountUp THEN
    (* Up counting *)
    TempValue := CurrentValue + StepSize;
    IF TempValue > MaxValue THEN
        CurrentValue := MaxValue; (* Clamp to MaxValue *)
        AtMax := TRUE;
        AtMin := FALSE;
    ELSE
        CurrentValue := TempValue;
        AtMax := (CurrentValue = MaxValue);
        AtMin := (CurrentValue = MinValue);
    END_IF;
ELSE
    (* Down counting *)
    TempValue := CurrentValue - StepSize;
    IF TempValue < MinValue THEN
        CurrentValue := MinValue; (* Clamp to MinValue *)
        AtMax := FALSE;
        AtMin := TRUE;
    ELSE
        CurrentValue := TempValue;
        AtMax := (CurrentValue = MaxValue);
        AtMin := (CurrentValue = MinValue);
    END_IF;
END_IF;

(* Step 6: Update Enable State *)
(* Track Enable for edge detection in next cycle *)
PreviousEnable := Enable;

END_FUNCTION_BLOCK
