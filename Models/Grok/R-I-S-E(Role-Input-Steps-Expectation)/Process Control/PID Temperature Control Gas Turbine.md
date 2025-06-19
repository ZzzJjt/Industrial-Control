VAR
    (* Inputs *)
    Temp_PV        : REAL;             (* Measured turbine inlet temperature [°C] *)
    Temp_SP        : REAL := 950.0;    (* Target temperature setpoint [°C] *)

    (* PID control variables *)
    Error          : REAL;             (* Current error [°C] *)
    Prev_Error     : REAL := 0.0;      (* Previous error for derivative term [°C] *)
    Integral       : REAL := 0.0;      (* Integral term accumulation *)
    Derivative     : REAL;             (* Derivative term *)
    Valve_Position : REAL;             (* Output to inlet valve [0-100%] *)

    (* PID tuning parameters *)
    Kp             : REAL := 3.0;      (* Proportional gain *)
    Ki             : REAL := 0.7;      (* Integral gain *)
    Kd             : REAL := 0.2;      (* Derivative gain *)

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
    Error := Temp_SP - Temp_PV;

    (* Update integral term *)
    Integral := Integral + Error * 0.1;  (* Sample time = 0.1 s *)

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / 0.1;  (* Sample time = 0.1 s *)

    (* Calculate PID output *)
    Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* Clamp valve position to safety limits *)
    IF Valve_Position > Valve_Max THEN
        Valve_Position := Valve_Max;
    ELSIF Valve_Position < Valve_Min THEN
        Valve_Position := Valve_Min;
    END_IF;

    (* Update previous error for next cycle *)
    Prev_Error := Error;

    (* Reset timer *)
    PID_Timer(IN := FALSE);
    PID_Timer(IN := TRUE);
END_IF;

(* Valve_Position is sent to the inlet valve *)
(* Example: Write Valve_Position to analog output for valve control *)
