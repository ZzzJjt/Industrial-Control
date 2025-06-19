VAR
    (* Inputs *)
    Pressure_PV    : REAL;             (* Measured reactor pressure [bar] *)
    Pressure_SP    : REAL := 5.0;      (* Target pressure setpoint [bar] *)

    (* PID control variables *)
    Error          : REAL;             (* Current error [bar] *)
    Prev_Error     : REAL := 0.0;      (* Previous error for derivative term [bar] *)
    Integral       : REAL := 0.0;      (* Integral term accumulation *)
    Derivative     : REAL;             (* Derivative term *)
    Valve_Output   : REAL;             (* Output to pressure control valve [0-100%] *)

    (* PID tuning parameters *)
    Kp             : REAL := 2.0;      (* Proportional gain *)
    Ki             : REAL := 0.8;      (* Integral gain *)
    Kd             : REAL := 0.3;      (* Derivative gain *)

    (* Safety limits *)
    Valve_Max      : REAL := 100.0;    (* Maximum valve position [%] *)
    Valve_Min      : REAL := 0.0;      (* Minimum valve position [%] *)

    (* Timing *)
    Sample_Time    : TIME := T#100ms;  (* Sampling period *)
    PID_Timer      : TON;              (* Timer for PID loop *)
END_VAR

(* Initialize timer for 100 ms sampling *)
PID_Timer(IN := TRUE, PT := Sample_Time);

(* PID control loop *)
IF PID_Timer.Q THEN
    (* Calculate error *)
    Error := Pressure_SP - Pressure_PV;

    (* Update integral term *)
    Integral := Integral + Error * 0.1;  (* Sample time = 0.1 s *)

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / 0.1;  (* Sample time = 0.1 s *)

    (* Calculate PID output *)
    Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* Clamp valve position to safety limits *)
    IF Valve_Output > Valve_Max THEN
        Valve_Output := Valve_Max;
    ELSIF Valve_Output < Valve_Min THEN
        Valve_Output := Valve_Min;
    END_IF;

    (* Update previous error for next cycle *)
    Prev_Error := Error;

    (* Reset timer *)
    PID_Timer(IN := FALSE);
    PID_Timer(IN := TRUE);
END_IF;

(* Valve_Output is sent to the pressure control valve *)
(* Example: Write Valve_Output to analog output for valve control *)
