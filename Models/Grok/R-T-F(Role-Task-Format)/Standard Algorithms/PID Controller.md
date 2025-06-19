(* IEC 61131-3 Structured Text function block for a PID controller *)
(* Implements proportional, integral, and derivative control with anti-windup *)
(* Optimized for single-cycle execution in real-time PLC environments *)

FUNCTION_BLOCK PIDController
VAR_INPUT
    Enable : BOOL; (* TRUE to enable PID execution *)
    Reset : BOOL; (* TRUE to reset internal states *)
    Setpoint : REAL; (* Desired process value *)
    ProcessVariable : REAL; (* Measured process value *)
    Kp : REAL := 1.0; (* Proportional gain *)
    Ki : REAL := 0.1; (* Integral gain *)
    Kd : REAL := 0.01; (* Derivative gain *)
    DeltaT : REAL := 0.1; (* Sample time in seconds *)
    MinOutput : REAL := -100.0; (* Minimum control output *)
    MaxOutput : REAL := 100.0; (* Maximum control output *)
END_VAR

VAR_OUTPUT
    ControlOutput : REAL; (* Calculated control signal *)
    Error : REAL; (* Current error: Setpoint - ProcessVariable *)
    IntegralTerm : REAL; (* Current integral term *)
    DerivativeTerm : REAL; (* Current derivative term *)
    Fault : BOOL := FALSE; (* TRUE if invalid inputs detected *)
END_VAR

VAR
    LastError : REAL; (* Previous error for derivative calculation *)
    LastIntegral : REAL; (* Previous integral term *)
    LastEnable : BOOL; (* Tracks previous Enable state for edge detection *)
    Initialized : BOOL := FALSE; (* Tracks if controller is initialized *)
END_VAR

(* Reset outputs when disabled *)
IF NOT Enable THEN
    ControlOutput := 0.0;
    Error := 0.0;
    IntegralTerm := 0.0;
    DerivativeTerm := 0.0;
    Fault := FALSE;
    Initialized := FALSE;
    LastEnable := FALSE;
    RETURN;
END_IF;

(* Input validation and initialization on rising edge of Enable *)
IF Enable AND NOT LastEnable THEN
    (* Validate inputs *)
    IF Kp < 0.0 OR Ki < 0.0 OR Kd < 0.0 OR DeltaT <= 0.0 OR MinOutput >= MaxOutput THEN
        Fault := TRUE;
        ControlOutput := 0.0;
        Error := 0.0;
        IntegralTerm := 0.0;
        DerivativeTerm := 0.0;
        Initialized := FALSE;
        RETURN;
    END_IF;
    
    (* Initialize states *)
    LastError := 0.0;
    LastIntegral := 0.0;
    Error := Setpoint - ProcessVariable;
    IntegralTerm := 0.0;
    DerivativeTerm := 0.0;
    ControlOutput := 0.0;
    Fault := FALSE;
    Initialized := TRUE;
END_IF;

(* Store Enable state for edge detection *)
LastEnable := Enable;

(* Reset handling *)
IF Reset AND Initialized THEN
    (* Reinitialize states *)
    LastError := 0.0;
    LastIntegral := 0.0;
    Error := Setpoint - ProcessVariable;
    IntegralTerm := 0.0;
    DerivativeTerm := 0.0;
    ControlOutput := 0.0;
    Fault := FALSE;
    RETURN;
END_IF;

(* PID calculation *)
IF Enable AND Initialized AND NOT Fault THEN
    (* Calculate error *)
    Error := Setpoint - ProcessVariable;
    
    (* Proportional term *)
    VAR P_term : REAL := Kp * Error; END_VAR
    
    (* Integral term with anti-windup *)
    IntegralTerm := LastIntegral;
    IF Ki > 0.0 THEN
        (* Only integrate if output is not saturated *)
        VAR u_unclamped : REAL := P_term + IntegralTerm + DerivativeTerm; END_VAR
        IF (u_unclamped >= MinOutput AND u_unclamped <= MaxOutput) OR 
           (u_unclamped < MinOutput AND Error > 0.0) OR 
           (u_unclamped > MaxOutput AND Error < 0.0) THEN
            IntegralTerm := IntegralTerm + Ki * Error * DeltaT;
        END_IF;
    END_IF;
    
    (* Derivative term *)
    DerivativeTerm := 0.0;
    IF Kd > 0.0 AND DeltaT > 0.0 AND Initialized THEN
        DerivativeTerm := Kd * (Error - LastError) / DeltaT;
    END_IF;
    
    (* Calculate control output *)
    ControlOutput := P_term + IntegralTerm + DerivativeTerm;
    
    (* Clamp output *)
    IF ControlOutput > MaxOutput THEN
        ControlOutput := MaxOutput;
    ELSIF ControlOutput < MinOutput THEN
        ControlOutput := MinOutput;
    END_IF;
    
    (* Update states for next cycle *)
    LastError := Error;
    LastIntegral := IntegralTerm;
END_IF;

(* Execution Notes *)
(* - Executes in a single scan cycle (<1 ms on modern PLCs), suitable for 10-50 ms scan times. *)
(* - Anti-windup prevents integral term growth when output is saturated, improving stability. *)
(* - Scalar arithmetic ensures compatibility with PLCs lacking matrix libraries. *)
(* - Input validation (Kp, Ki, Kd >= 0, DeltaT > 0, MinOutput < MaxOutput) prevents numerical issues. *)
(* - Outputs Error, IntegralTerm, DerivativeTerm for diagnostics and tuning. *)
END_FUNCTION_BLOCK
