(* IEC 61131-3 Structured Text: PID Controller Function Block *)
(* Purpose: Implements a standard PID controller for precise process control *)

FUNCTION_BLOCK PIDController
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run controller *)
    Reset : BOOL;                   (* TRUE to reset integral and error *)
    Setpoint : REAL;                (* Desired process value *)
    ProcessVariable : REAL;         (* Measured process value *)
    Kp : REAL;                      (* Proportional gain *)
    Ki : REAL;                      (* Integral gain *)
    Kd : REAL;                      (* Derivative gain *)
    DeltaT : REAL;                  (* Sample time in seconds, e.g., 0.1 *)
    MinOutput : REAL;               (* Minimum control output *)
    MaxOutput : REAL;               (* Maximum control output *)
END_VAR
VAR_OUTPUT
    ControlOutput : REAL;           (* Control signal output *)
    Error : REAL;                   (* Current error: Setpoint - ProcessVariable *)
    IntegralTerm : REAL;            (* Current integral term *)
    DerivativeTerm : REAL;          (* Current derivative term *)
    OutputError : BOOL;             (* TRUE if input validation fails *)
END_VAR
VAR
    PrevError : REAL := 0.0;        (* Previous error for derivative calculation *)
    Integral : REAL := 0.0;         (* Accumulated integral term *)
    Initialized : BOOL := FALSE;    (* TRUE after first run or reset *)
END_VAR

(* Main Logic *)
IF Reset THEN
    (* Reset internal states *)
    Initialized := FALSE;
END_IF;

IF NOT Enable THEN
    (* Disable controller, reset outputs *)
    ControlOutput := 0.0;
    Error := 0.0;
    IntegralTerm := 0.0;
    DerivativeTerm := 0.0;
    OutputError := FALSE;
ELSE
    (* Input Validation *)
    IF DeltaT <= 0.0 OR Kp < 0.0 OR Ki < 0.0 OR Kd < 0.0 OR MinOutput > MaxOutput THEN
        (* Invalid parameters: DeltaT, gains, or output bounds *)
        OutputError := TRUE;
        ControlOutput := 0.0;
        Error := 0.0;
        IntegralTerm := 0.0;
        DerivativeTerm := 0.0;
    ELSE
        OutputError := FALSE;
        
        (* Initialize on first run or after reset *)
        IF NOT Initialized THEN
            PrevError := 0.0;
            Integral := 0.0;
            Initialized := TRUE;
        END_IF;
        
        (* Calculate Error *)
        Error := Setpoint - ProcessVariable;
        
        (* Proportional Term *)
        P_term := Kp * Error;
        
        (* Integral Term with Anti-Windup *)
        Integral := Integral + Ki * Error * DeltaT;
        
        (* Derivative Term *)
        IF DeltaT > 0.0 THEN
            DerivativeTerm := Kd * (Error - PrevError) / DeltaT;
        ELSE
            DerivativeTerm := 0.0;  (* Avoid division by zero *)
        END_IF;
        
        (* Compute Control Output *)
        ControlOutput := P_term + Integral + DerivativeTerm;
        
        (* Output Clamping and Anti-Windup *)
        IF ControlOutput > MaxOutput THEN
            ControlOutput := MaxOutput;
            IF Ki > 0.0 THEN
                (* Back-calculate integral to prevent windup *)
                Integral := Integral - Ki * Error * DeltaT;
            END_IF;
        ELSIF ControlOutput < MinOutput THEN
            ControlOutput := MinOutput;
            IF Ki > 0.0 THEN
                (* Back-calculate integral to prevent windup *)
                Integral := Integral - Ki * Error * DeltaT;
            END_IF;
        END_IF;
        
        (* Update Integral Term for Output *)
        IntegralTerm := Integral;
        
        (* Store Current Error for Next Cycle *)
        PrevError := Error;
    END_IF;
END_IF;

(* Notes:
   - Purpose: Implements a PID controller for precise control of temperature, pressure, flow, or position.
   - Algorithm: u(t) = Kp * e(t) + Ki * ∫e(t)dt + Kd * de(t)/dt
   - Inputs:
     - Enable: Runs controller when TRUE.
     - Reset: Resets integral and previous error when TRUE.
     - Setpoint: Desired process value (e.g., 100.0°C).
     - ProcessVariable: Measured value (e.g., 95.0°C).
     - Kp: Proportional gain (e.g., 2.0).
     - Ki: Integral gain (e.g., 0.1).
     - Kd: Derivative gain (e.g., 0.5).
     - DeltaT: Sample time (s, e.g., 0.1).
     - MinOutput: Minimum output (e.g., 0.0).
     - MaxOutput: Maximum output (e.g., 100.0).
   - Outputs:
     - ControlOutput: Control signal (e.g., valve position, %).
     - Error: Current error (Setpoint - ProcessVariable).
     - IntegralTerm: Current integral term for monitoring.
     - DerivativeTerm: Current derivative term for monitoring.
     - OutputError: TRUE if DeltaT <= 0, gains < 0, or MinOutput > MaxOutput.
   - Features:
     - Tunable Kp, Ki, Kd for flexible control tuning.
     - Anti-windup via back-calculation when output is clamped.
     - Output clamping to MinOutput..MaxOutput.
     - Reset logic clears integral and error.
   - Safety:
     - Validates inputs to prevent division by zero or invalid operations.
     - Clamps output to avoid exceeding actuator limits.
     - Resets outputs on disable or error for safe state.
   - Usage:
     - Suitable for HVAC, thermal processing, or motion control in PLC systems.
     - Reusable in scan-cycle-driven environments (e.g., CODESYS, TwinCAT).
   - Maintenance:
     - Scalar math ensures PLC compatibility.
     - Comments and modular design aid debugging.
     - Error, IntegralTerm, and DerivativeTerm support diagnostics via HMI.
   - Platform Notes:
     - Uses REAL for calculations; LREAL can be used for higher precision.
     - Assumes single-cycle execution; multi-cycle logic can be added for slow PLCs.
*)
END_FUNCTION_BLOCK
