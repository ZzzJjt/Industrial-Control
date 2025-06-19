(* IEC 61131-3 Structured Text Function Block for PID Controller *)
(* Implements a discrete-time PID controller for real-time control *)
(* Includes anti-windup, output clamping, and input validation *)
(* Conforms to industrial automation best practices *)

FUNCTION_BLOCK PIDController
VAR_INPUT
    Setpoint: REAL;            (* Desired process value, e.g., 100°C *)
    ProcessVariable: REAL;     (* Measured process value, e.g., current temperature *)
    Kp: REAL;                  (* Proportional gain, e.g., 2.0 *)
    Ki: REAL;                  (* Integral gain, e.g., 0.1 *)
    Kd: REAL;                  (* Derivative gain, e.g., 0.05 *)
    MinOutput: REAL;           (* Minimum control output, e.g., 0.0 *)
    MaxOutput: REAL;           (* Maximum control output, e.g., 100.0 *)
    DeltaT: REAL;              (* Sample time in seconds, e.g., 0.1 *)
    Enable: BOOL;              (* TRUE to activate controller *)
    Reset: BOOL;               (* TRUE to reset internal states *)
END_VAR

VAR_OUTPUT
    ControlOutput: REAL;       (* Computed control signal, e.g., valve position *)
    ValidOutput: BOOL;         (* TRUE if output is valid *)
END_VAR

VAR
    (* Internal state *)
    Integral: REAL;            (* Accumulated integral term *)
    PrevProcessVariable: REAL; (* Previous process variable for derivative *)
    PrevError: REAL;           (* Previous error for continuity *)
    
    (* Temporary variables *)
    Error: REAL;               (* Current error: Setpoint - ProcessVariable *)
    P_term: REAL;              (* Proportional term *)
    I_term: REAL;              (* Integral term *)
    D_term: REAL;              (* Derivative term *)
    RawOutput: REAL;           (* Unclamped output *)
    
    (* Validation *)
    ValidInput: BOOL;          (* TRUE if inputs are valid *)
    Initialized: BOOL;         (* TRUE after first valid cycle *)
END_VAR

(* Step 1: Handle Reset *)
(* Reset internal states when Reset is TRUE *)
IF Reset THEN
    Integral := 0.0;
    PrevProcessVariable := 0.0;
    PrevError := 0.0;
    ControlOutput := 0.0;
    ValidOutput := FALSE;
    Initialized := FALSE;
    RETURN;
END_IF;

(* Step 2: Input Validation *)
(* Ensure DeltaT is positive, gains are non-negative, and output bounds are valid *)
ValidInput := (DeltaT > 0.0) AND (Kp >= 0.0) AND (Ki >= 0.0) AND (Kd >= 0.0) 
              AND (MinOutput <= MaxOutput) AND (ABS(ProcessVariable) < 1.0E6) 
              AND (ABS(Setpoint) < 1.0E6);

(* Step 3: Check Enable and Valid Input *)
(* Exit if not enabled or inputs are invalid *)
IF NOT Enable OR NOT ValidInput THEN
    ControlOutput := 0.0;
    ValidOutput := FALSE;
    RETURN;
END_IF;

(* Step 4: Initialize on First Cycle *)
(* Set initial states after first valid measurement *)
IF NOT Initialized THEN
    Integral := 0.0;
    PrevProcessVariable := ProcessVariable;
    PrevError := Setpoint - ProcessVariable;
    ControlOutput := 0.0;
    ValidOutput := TRUE;
    Initialized := TRUE;
    RETURN;
END_IF;

(* Step 5: Compute Error *)
(* Error = Setpoint - ProcessVariable *)
Error := Setpoint - ProcessVariable;

(* Step 6: Proportional Term *)
(* P = Kp * Error *)
P_term := Kp * Error;

(* Step 7: Integral Term with Anti-Windup *)
(* I = I_prev + Ki * Error * DeltaT, clamped to prevent windup *)
I_term := Integral + Ki * Error * DeltaT;

(* Step 8: Derivative Term *)
(* D = Kd * (PrevProcessVariable - ProcessVariable) / DeltaT *)
(* Use process variable rate to avoid setpoint kick *)
IF DeltaT > 0.0 THEN
    D_term := Kd * (PrevProcessVariable - ProcessVariable) / DeltaT;
ELSE
    D_term := 0.0;  (* Avoid division by zero *)
END_IF;

(* Step 9: Compute Raw Output *)
(* RawOutput = P + I + D *)
RawOutput := P_term + I_term + D_term;

(* Step 10: Clamp Output and Apply Anti-Windup *)
(* Clamp to MinOutput and MaxOutput, adjust integral if saturated *)
IF RawOutput > MaxOutput THEN
    ControlOutput := MaxOutput;
    (* Anti-windup: Only update integral if it reduces saturation *)
    IF I_term > 0.0 THEN
        I_term := Integral;  (* Prevent integral growth *)
    END_IF;
ELSIF RawOutput < MinOutput THEN
    ControlOutput := MinOutput;
    IF I_term < 0.0 THEN
        I_term := Integral;  (* Prevent integral reduction *)
    END_IF;
ELSE
    ControlOutput := RawOutput;
END_IF;

(* Step 11: Update Integral *)
(* Only update integral if output is not saturated *)
Integral := I_term;

(* Step 12: Update Previous Values *)
(* Store current values for next cycle *)
PrevProcessVariable := ProcessVariable;
PrevError := Error;

(* Step 13: Set Output Validity *)
(* Output is valid if processing completed *)
ValidOutput := TRUE;

(* Step 14: Safety Check *)
(* Ensure output and internal states are reasonable *)
IF ABS(ControlOutput) > 1.0E6 OR ABS(Integral) > 1.0E6 THEN
    ControlOutput := 0.0;
    Integral := 0.0;
    ValidOutput := FALSE;
END_IF;

END_FUNCTION_BLOCK
