VAR
    (* Primary loop: Pressure control *)
    PV1            : REAL;             (* Measured vessel pressure [bar] *)
    SP1            : REAL;             (* Target pressure setpoint [bar] *)
    e1             : REAL;             (* Pressure error *)
    Prev_e1        : REAL := 0.0;      (* Previous pressure error *)
    Integral1      : REAL := 0.0;      (* Integral term for pressure *)
    Derivative1    : REAL;             (* Derivative term for pressure *)
    OP1            : REAL;             (* Output: flow setpoint [L/min] *)
    Kp1            : REAL;             (* Proportional gain for pressure *)
    Ki1            : REAL;             (* Integral gain for pressure *)
    Kd1            : REAL;             (* Derivative gain for pressure *)

    (* Secondary loop: Flow control *)
    PV2            : REAL;             (* Measured flow rate [L/min] *)
    SP2            : REAL;             (* Target flow setpoint [L/min], set by OP1 *)
    e2             : REAL;             (* Flow error *)
    Prev_e2        : REAL := 0.0;      (* Previous flow error *)
    Integral2      : REAL := 0.0;      (* Integral term for flow *)
    Derivative2    : REAL;             (* Derivative term for flow *)
    OP2            : REAL;             (* Output: valve position [0-100%] *)
    Kp2            : REAL;             (* Proportional gain for flow *)
    Ki2            : REAL;             (* Integral gain for flow *)
    Kd2            : REAL;             (* Derivative gain for flow *)

    (* Timing *)
    dt             : REAL := 0.1;      (* Sample time [s] *)
    Sample_Time    : TIME := T#100ms;  (* Sampling period *)
    PID_Timer      : TON;              (* Timer for PID loops *)
END_VAR

(* Initialize timer for 100 ms sampling *)
PID_Timer(IN := TRUE, PT := Sample_Time);

(* Cascade PID control loop *)
IF PID_Timer.Q THEN
    (* Read measurements *)
    PV1 := ReadPressure();
    PV2 := ReadFlowRate();

    (* Primary loop: Pressure control *)
    e1 := SP1 - PV1;  (* Calculate pressure error *)
    Integral1 := Integral1 + e1 * dt;  (* Update integral *)
    Derivative1 := (e1 - Prev_e1) / dt;  (* Calculate derivative *)
    OP1 := (Kp1 * e1) + (Ki1 * Integral1) + (Kd1 * Derivative1);  (* PID output *)
    
    (* Clamp OP1 *)
    IF OP1 > 100.0 THEN
        OP1 := 100.0;
    ELSIF OP1 < 0.0 THEN
        OP1 := 0.0;
    END_IF;
    Prev_e1 := e1;  (* Store error *)

    (* Secondary loop: Flow control *)
    SP2 := OP1;  (* Set flow setpoint *)
    e2 := SP2 - PV2;  (* Calculate flow error *)
    Integral2 := Integral2 + e2 * dt;  (* Update integral *)
    Derivative2 := (e2 - Prev_e2) / dt;  (* Calculate derivative *)
    OP2 := (Kp2 * e2) + (Ki2 * Integral2) + (Kd2 * Derivative2);  (* PID output *)
    
    (* Clamp OP2 *)
    IF OP2 > 100.0 THEN
        OP2 := 100.0;
    ELSIF OP2 < 0.0 THEN
        OP2 := 0.0;
    END_IF;
    Prev_e2 := e2;  (* Store error *)

    (* Send output to valve *)
    SetValvePosition(OP2);

    (* Reset timer *)
    PID_Timer(IN := FALSE);
    PID_Timer(IN := TRUE);
END_IF;
