FUNCTION_BLOCK FB_DigitalCounter
VAR_INPUT
    Enable : BOOL;              // Enables counting when TRUE
    CountUp : BOOL;             // TRUE for up counting, FALSE for down counting
    StepSize : INT := 1;        // Positive step size for counting
    InitValue : INT := 0;       // Initial counter value on reset
    Reset : BOOL;               // Resets counter to InitValue when TRUE
    MaxValue : INT := 32767;    // Upper limit for counter
    MinValue : INT := -32768;   // Lower limit for counter
END_VAR
VAR_OUTPUT
    CurrentValue : INT;         // Current counter value
    AtMax : BOOL;               // TRUE when CurrentValue reaches MaxValue
    AtMin : BOOL;               // TRUE when CurrentValue reaches MinValue
END_VAR
VAR
    ValidInputs : BOOL := TRUE; // Tracks input validity
END_VAR

(* Counter logic with input validation and boundary handling *)
(* Execute in a single scan cycle for PLC compatibility *)

(* Input validation *)
IF StepSize <= 0 OR MinValue >= MaxValue OR 
   InitValue < MinValue OR InitValue > MaxValue THEN
    ValidInputs := FALSE;
    CurrentValue := 0;
    AtMax := FALSE;
    AtMin := FALSE;
    RETURN;
ELSE
    ValidInputs := TRUE;
END_IF;

(* Reset handling: Takes priority *)
IF Reset THEN
    CurrentValue := InitValue;
    AtMax := CurrentValue >= MaxValue;
    AtMin := CurrentValue <= MinValue;
    RETURN;
END_IF;

(* Counting logic: Only when Enable = TRUE and inputs are valid *)
IF Enable AND ValidInputs THEN
    IF CountUp THEN
        (* Up counting *)
        IF CurrentValue <= MaxValue - StepSize THEN
            CurrentValue := CurrentValue + StepSize;
        ELSE
            CurrentValue := MaxValue; // Clamp at MaxValue
        END_IF;
    ELSE
        (* Down counting *)
        IF CurrentValue >= MinValue + StepSize THEN
            CurrentValue := CurrentValue - StepSize;
        ELSE
            CurrentValue := MinValue; // Clamp at MinValue
        END_IF;
    END_IF;
ELSE
    (* Maintain current value when disabled *)
    CurrentValue := CurrentValue;
END_IF;

(* Update boundary flags *)
AtMax := CurrentValue >= MaxValue;
AtMin := CurrentValue <= MinValue;

END_FUNCTION_BLOCK
```

### Explanation of Implementation
1. **Function Block Interface**:
   - **Inputs**:
     - `Enable`: Triggers counting when `TRUE`.
     - `CountUp`: Selects up (`TRUE`) or down (`FALSE`) counting.
     - `StepSize`: Positive `INT` for increment/decrement (default 1).
     - `InitValue`: Starting value on reset (default 0).
     - `Reset`: Resets to `InitValue` when `TRUE`.
     - `MaxValue`: Upper limit (default 32767, max `INT`).
     - `MinValue`: Lower limit (default -32768, min `INT`).
   - **Outputs**:
     - `CurrentValue`: Current counter value, updated based on counting logic.
     - `AtMax`: `TRUE` when `CurrentValue >= MaxValue`.
     - `AtMin`: `TRUE` when `CurrentValue <= MinValue`.
   - **Internal Variable**:
     - `ValidInputs`: Tracks input validity to prevent erroneous counting.

2. **Internal Logic**:
   - **Input Validation**:
     - Checks `StepSize > 0` to ensure valid increments.
     - Ensures `MinValue < MaxValue` for a valid range.
     - Verifies `MinValue <= InitValue <= MaxValue` for a valid starting point.
     - If invalid, sets `ValidInputs := FALSE`, resets `CurrentValue := 0`, and clears flags.
   - **Reset Handling**:
     - When `Reset = TRUE`, sets `CurrentValue := InitValue` and updates `AtMax`/`AtMin` based on boundaries.
     - Takes priority over counting to ensure immediate reinitialization.
   - **Counting Logic**:
     - Executes only when `Enable = TRUE` and `ValidInputs = TRUE`.
     - **Up Counting** (`CountUp = TRUE`):
       - Increments `CurrentValue` by `StepSize` if `CurrentValue <= MaxValue - StepSize`.
       - Clamps to `MaxValue` if the next step would exceed it, preventing overflow.
     - **Down Counting** (`CountUp = FALSE`):
       - Decrements `CurrentValue` by `StepSize` if `CurrentValue >= MinValue + StepSize`.
       - Clamps to `MinValue` if the next step would go below it, preventing underflow.
     - When `Enable = FALSE`, maintains `CurrentValue` to preserve state.
   - **Boundary Flags**:
     - `AtMax := CurrentValue >= MaxValue`, indicating the upper limit is reached.
     - `AtMin := CurrentValue <= MinValue`, indicating the lower limit is reached.

3. **Outputs**:
   - `CurrentValue`: Reflects the current counter state, updated per counting or reset logic.
   - `AtMax` and `AtMin`: Provide status flags for external logic (e.g., stopping a conveyor when `AtMax = TRUE`).
   - Outputs are deterministic and maintain values between scan cycles, per PLC conventions.

4. **Comments and Safeguards**:
   - **Comments**: Detailed inline explanations cover validation, reset, counting, and boundary handling, aiding maintenance and onboarding.
   - **Safeguards**:
     - Input validation prevents invalid configurations (e.g., negative `StepSize`, `MinValue > MaxValue`).
     - Overflow/underflow protection clamps `CurrentValue` to `MaxValue` or `MinValue`.
     - `ValidInputs` ensures safe operation by disabling counting on error.
     - `Reset` priority ensures predictable reinitialization.
   - **Scan-Cycle Safety**: The logic uses simple conditionals (`IF`) with no loops, completing in <1 ms on typical PLCs, well within a 10–100 ms scan cycle.

### Meeting Expectations
- **Robustness**:
  - **Scan-Cycle Stability**: Executes in a single scan cycle with minimal operations (conditionals and arithmetic), ensuring real-time compatibility.
  - **Error Handling**: Validates inputs (`StepSize`, `MinValue`, `MaxValue`, `InitValue`) and clamps `CurrentValue` to prevent overflow/underflow.
  - **Reliability**: Deterministic behavior with clear state transitions (reset, count, hold) and boundary flags (`AtMax`, `AtMin`).
- **Modularity**:
  - Encapsulated in `FB_DigitalCounter`, with a clear interface (`Enable`, `CurrentValue`, etc.) for easy integration.
  - Can be instantiated multiple times for different counting tasks (e.g., parts counting, batch tracking).
- **Reusability**:
  - Generic design supports various applications (e.g., motor steps, production counts) by configuring `StepSize`, `InitValue`, `MaxValue`, and `MinValue`.
  - Adaptable to other data types (e.g., `DINT`, `REAL`) by changing variable declarations, with minor logic adjustments for floating-point comparisons.
- **Ease of Use**:
  - Intuitive inputs (`Enable`, `CountUp`, `Reset`) and outputs (`CurrentValue`, `AtMax`, `AtMin`) align with industrial control patterns.
  - Default values (`StepSize = 1`, `InitValue = 0`) simplify initial use.
- **Maintainability**:
  - Detailed comments explain each control mechanism, validation, and safeguard, aiding debugging and onboarding.
  - Clear variable names (e.g., `StepSize`, `AtMax`) and structured logic enhance readability.
- **Scalability**:
  - Fixed limits (`MaxValue = 32767`, `MinValue = -32768`) cover typical `INT` use cases but can be adjusted for specific needs.
  - For dynamic ranges or larger arrays, the logic can be extended with `ANY_INT` or custom bounds, though this requires vendor-specific support (e.g., CODESYS).

### Additional Notes
- **Performance**: The function block uses simple conditionals, completing in <1 ms on typical PLCs, ensuring scan-cycle safety even in tight cycles (10 ms). No loops or complex computations are involved.
- **Validation Trade-Off**: Comprehensive input validation (`StepSize > 0`, `MinValue < MaxValue`) ensures robustness but adds minor overhead. For performance-critical applications, validation can be simplified if inputs are guaranteed valid.
- **Adaptability**: 
  - To support dynamic ranges, add inputs for `ArraySize` and use `ANY_INT` with bounds checking, though this may require platform-specific features.
  - For non-integer counting (e.g., `REAL`), modify comparisons and clamping logic to handle floating-point precision.
- **Safety**: The block is stateless except for `CurrentValue`, which is deterministically updated, avoiding issues in cyclic execution. Add explicit fault codes (e.g., `Fault_Code : INT`) for production if needed.
- **Compliance**: Aligns with IEC 61131-3 standards for function blocks, ensuring portability across platforms (e.g., CODESYS, TwinCAT, Siemens).
- **Use Case Example**: Counting parts on a conveyor (`StepSize = 1`, `MaxValue = 1000`) with `AtMax` triggering a batch stop, or tracking motor steps (`StepSize = 10`, `MinValue = 0`) with `Reset` on cycle completion.

This function block is ready for deployment in a real-time PLC environment, providing a robust and reusable digital counter utility. If you need extensions (e.g., dynamic ranges, additional features like pause/resume, or specific PLC platform optimizations), please provide details, and I can refine the code further!
