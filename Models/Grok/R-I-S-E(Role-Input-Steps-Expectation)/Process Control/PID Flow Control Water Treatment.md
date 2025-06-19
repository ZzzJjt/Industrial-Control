VAR
    (* Inputs *)
    FlowRate       : REAL;             (* Measured flow rate [L/min] *)
    Dosing_PV      : REAL;             (* Measured chlorine concentration [ppm] *)
    Dosing_SP      : REAL := 3.0;      (* Target chlorine concentration [ppm] *)

    (* PID control variables *)
    Error          : REAL;             (* Current error [ppm] *)
    Prev_Error     : REAL := 0.0;      (* Previous error for derivative term [ppm] *)
    Integral       : REAL := 0.0;      (* Integral term accumulation *)
    Derivative     : REAL;             (* Derivative term *)
    Dosing_Output  : REAL;             (* Output to dosing pump [0-10] *)

    (* PID tuning parameters *)
    Kp             : REAL := 2.0;      (* Proportional gain *)
    Ki             : REAL := 0.5;      (* Integral gain *)
    Kd             : REAL := 0.1;      (* Derivative gain *)

    (* Safety limits *)
    Max_Dose       : REAL := 10.0;     (* Maximum dosing output *)
    Min_Dose       : REAL := 0.0;      (* Minimum dosing output *)

    (* Timing *)
    Sample_Time    : TIME := T#100ms;  (* Sampling period *)
    PID_Timer      : TON;              (* Timer for PID loop *)
END_VAR

(* Initialize timer for 100 ms sampling *)
PID_Timer(IN := TRUE, PT := Sample_Time);

(* PID control loop *)
IF PID_Timer.Q THEN
    (* Calculate error *)
    Error := Dosing_SP - Dosing_PV;

    (* Update integral term *)
    Integral := Integral + Error * 0.1;  (* Sample time = 0.1 s *)

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / 0.1;  (* Sample time = 0.1 s *)

    (* Calculate PID output *)
    Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* Clamp output to safety limits *)
    IF Dosing_Output > Max_Dose THEN
        Dosing_Output := Max_Dose;
    ELSIF Dosing_Output < Min_Dose THEN
        Dosing_Output := Min_Dose;
    END_IF;

    (* Update previous error for next cycle *)
    Prev_Error := Error;

    (* Reset timer *)
    PID_Timer(IN := FALSE);
    PID_Timer(IN := TRUE);
END_IF;

(* Dosing_Output is sent to the dosing pump *)
(* Example: Write Dosing_Output to analog output for pump control *)
