VAR
    (* Outer loop (temperature control) *)
    Temp_SP         : REAL := 85.0;      (* Temperature setpoint [°C] *)
    Temp_PV         : REAL;             (* Measured temperature [°C] *)
    Temp_Error      : REAL;             (* Temperature error [°C] *)
    Flow_SP         : REAL;             (* Flow setpoint for inner loop [l/min] *)

    (* Inner loop (flow control) *)
    Flow_PV         : REAL;             (* Measured flow rate [l/min] *)
    Flow_Error      : REAL;             (* Flow error [l/min] *)
    Flow_Output     : REAL;             (* Control output to valve [0-100%] *)

    (* Control parameters *)
    Kp_Outer        : REAL := 1.0;      (* Proportional gain for outer loop *)
    Kp_Inner        : REAL := 2.0;      (* Proportional gain for inner loop *)

    (* Timing for cascade control *)
    Outer_Loop_Time : TIME := T#500ms;  (* Outer loop update interval *)
    Inner_Loop_Time : TIME := T#100ms;  (* Inner loop update interval, 5x faster *)
    Outer_Timer     : TON;              (* Timer for outer loop *)
    Inner_Timer     : TON;              (* Timer for inner loop *)
END_VAR

(* Initialize timers for cascade loop timing *)
Outer_Timer(IN := TRUE, PT := Outer_Loop_Time);
Inner_Timer(IN := TRUE, PT := Inner_Loop_Time);

(* Outer loop: Temperature controller *)
IF Outer_Timer.Q THEN
    (* Calculate temperature error *)
    Temp_Error := Temp_SP - Temp_PV;
    
    (* Generate flow setpoint for inner loop *)
    Flow_SP := Kp_Outer * Temp_Error;
    
    (* Limit Flow_SP to realistic bounds [0 to 100 l/min] *)
    IF Flow_SP > 100.0 THEN
        Flow_SP := 100.0;
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

(* Flow_Output is sent to the flow control valve *)
(* Example: Analog output channel would be written with Flow_Output *)
