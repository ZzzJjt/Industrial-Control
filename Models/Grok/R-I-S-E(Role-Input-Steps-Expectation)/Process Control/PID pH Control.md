VAR
    (* Inputs *)
    pH_PV          : REAL;             (* Measured pH value *)
    pH_SP          : REAL := 7.0;      (* Target pH setpoint *)

    (* PID control variables *)
    Error          : REAL;             (* Current error *)
    Prev_Error     : REAL := 0.0;      (* Previous error for derivative term *)
    Integral       : REAL := 0.0;      (* Integral term accumulation *)
    Derivative     : REAL;             (* Derivative term *)
    Dosing_Output  : REAL;             (* Output to dosing pump/valve [0-100%] *)

    (* PID tuning parameters *)
    Kp             : REAL := 2.5;      (* Proportional gain *)
    Ki             : REAL := 0.6;      (* Integral gain *)
    Kd             : REAL := 0.3;      (* Derivative gain *)

    (* Safety limits *)
    Dosing_Max     : REAL := 100.0;    (* Maximum dosing output [%] *)
    Dosing_Min     : REAL := 0.0;      (* Minimum dosing output [%] *)

    (* Timing *)
    Sample_Time    : TIME := T#100ms;  (* Sampling period *)
    PID_Timer      : TON;              (* Timer for PID loop *)
END_VAR

(* Initialize timer for 100 ms sampling *)
PID_Timer(IN := TRUE, PT := Sample_Time);

(* PID control loop *)
IF PID_Timer.Q THEN
    (* Calculate error *)
    Error := pH_SP - pH_PV;

    (* Update integral term *)
    Integral := Integral + Error * 0.1;  (* Sample time = 0.1 s *)

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / 0.1;  (* Sample time = 0.1 s *)

    (* Calculate PID output *)
    Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* Clamp dosing output to safety limits *)
    IF Dosing_Output > Dosing_Max THEN
        Dosing_Output := Dosing_Max;
    ELSIF Dosing_Output < Dosing_Min THEN
        Dosing_Output := Dosing_Min;
    END_IF;

    (* Update previous error for next cycle *)
    Prev_Error := Error;

    (* Reset timer *)
    PID_Timer(IN := FALSE);
    PID_Timer(IN := TRUE);
END_IF;

(* Dosing_Output is sent to the acid/base dosing pump or valve *)
(* Example: Write Dosing_Output to analog output for dosing control *)
