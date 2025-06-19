VAR
    (* Outer loop: Pressure control *)
    Pressure_SP     : REAL := 12.0;     (* Pressure setpoint [bar] *)
    Pressure_PV     : REAL;             (* Measured pressure [bar] *)
    Pressure_Error  : REAL;             (* Pressure error [bar] *)
    Flow_SP         : REAL;             (* Flow setpoint for innerSw loop [m³/h] *)

    (* Inner loop: Flow control *)
    Flow_PV         : REAL;             (* Measured flow rate [m³/h] *)
    Flow_Error      : REAL;             (* Flow error [m³/h] *)
    Flow_Output     : REAL;             (* Control output to valve [0-100%] *)

    (* Controller gains *)
    Kp_Outer        : REAL := 1.2;      (* Proportional gain for outer loop *)
    Kp_Inner        : REAL := 2.5;      (* Proportional gain for inner loop *)

    (* Timing for cascade control *)
    Outer_Loop_Time : TIME := T#1s;     (* Outer loop update interval *)
    Inner_Loop_Time : TIME := T#200ms;  (* Inner loop update interval, 5x faster *)
    Outer_Timer     : TON;              (* Timer for outer loop *)
    Inner_Timer     : TON;              (* Timer for inner loop *)
END_VAR

(* Initialize timers for cascade loop timing *)
Outer_Timer(IN := TRUE, PT := Outer_Loop_Time);
Inner_Timer(IN := TRUE, PT := Inner_Loop_Time);

(* Outer loop: Pressure controller *)
IF Outer_Timer.Q THEN
    (* Calculate pressure error *)
    Pressure_Error := Pressure_SP - Pressure_PV;
    
    (* Generate flow setpoint for inner loop *)
    Flow_SP := Kp_Outer * Pressure_Error;
    
    (* Limit Flow_SP to realistic bounds [0 to 50 m³/h] *)
    IF Flow_SP > 50.0 THEN
        Flow_SP := 50.0;
    ELSIF Flow_SP < 0.0 THEN
        Flow_SP := 0.0;
    END_IF;
    
    (* Reset outer loop timer *)
    Outer_Timer(IN := FALSE);
    Outer_Timer(IN := TRUE);
END_IF;

(* Inner loop: Flow controller *)
IF Inner_Timer.Q THEN
    (* Calculate flow error *)
    Flow_Error := Flow_SP - Flow_PV;
    
    (* Generate valve control output *)
    Flow_Output := Kp_Inner * Flow_Error;
    
    (* Limit Flow_Output to valve range [0 to 100%] *)
    IF Flow_Output > 100.0 THEN
        Flow_Output := 100.0;
    ELSIF Flow_Output < 0.0 THEN
        Flow_Output := 0.0;
    END_IF;
    
    (* Reset inner loop timer *)
    Inner_Timer(IN := FALSE);
    Inner_Timer(IN := TRUE);
END_IF;

(* Flow_Output is sent to the oil control valve *)
(* Example: Write Flow_Output to analog output for valve control *)
