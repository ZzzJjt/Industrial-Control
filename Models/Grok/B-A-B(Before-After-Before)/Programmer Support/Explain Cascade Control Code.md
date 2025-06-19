(* Vessel Cascade Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain stable vessel pressure using cascade PID control with inner flow loop *)

PROGRAM VesselCascadeControl
VAR
    (* Outer Loop Inputs (Pressure) *)
    Pressure_SP : REAL := 10.0;     (* Setpoint for vessel pressure in bar *)
    Pressure_PV : REAL;             (* Measured vessel pressure in bar *)

    (* Inner Loop Inputs (Flow) *)
    Flow_PV : REAL;                 (* Measured flow rate in L/min *)
    Flow_SP : REAL;                 (* Setpoint for flow rate, from outer loop output *)

    (* Control Outputs *)
    Valve_Output : REAL;            (* Control signal to valve, 0.0 to 100.0% *)

    (* Outer Loop PID Variables *)
    Pressure_Error : REAL;          (* Error for pressure loop *)
    Pressure_Integral : REAL := 0.0; (* Integral term for pressure loop *)
    Pressure_Derivative : REAL;     (* Derivative term for pressure loop *)
    Pressure_Prev_Error : REAL := 0.0; (* Previous error for derivative *)
    Pressure_Output : REAL;         (* Output of pressure loop *)

    (* Inner Loop PID Variables *)
    Flow_Error : REAL;              (* Error for flow loop *)
    Flow_Integral : REAL := 0.0;    (* Integral term for flow loop *)
    Flow_Derivative : REAL;         (* Derivative term for flow loop *)
    Flow_Prev_Error : REAL := 0.0;  (* Previous error for derivative *)
    Flow_Output : REAL;             (* Output of flow loop *)

    (* Tuning Parameters *)
    Kp1 : REAL := 1.5;              (* Proportional gain for pressure loop *)
    Ki1 : REAL := 0.3;              (* Integral gain for pressure loop *)
    Kd1 : REAL := 0.1;              (* Derivative gain for pressure loop *)
    Kp2 : REAL := 2.0;              (* Proportional gain for flow loop *)
    Ki2 : REAL := 0.5;              (* Integral gain for flow loop *)
    Kd2 : REAL := 0.2;              (* Derivative gain for flow loop *)
    CycleTime : REAL := 0.1;        (* Sample time in seconds, 100 ms *)

    (* Safety Limits *)
    Max_Valve : REAL := 100.0;      (* Maximum valve opening in % *)
    Min_Valve : REAL := 0.0;        (* Minimum valve opening in % *)
    Max_Flow_SP : REAL := 100.0;    (* Maximum flow setpoint in L/min *)
    Min_Flow_SP : REAL := 0.0;      (* Minimum flow setpoint in L/min *)
    Max_Pressure : REAL := 12.0;    (* Maximum allowable pressure in bar *)
    Min_Pressure : REAL := 8.0;     (* Minimum allowable pressure in bar *)
    Pressure_Fault : BOOL;          (* TRUE if pressure input is invalid *)
END_VAR

(* Main Control Logic *)
(* 1. Input Validation *)
IF Pressure_PV < Min_Pressure OR Pressure_PV > Max_Pressure THEN
    Pressure_Fault := TRUE;
    Valve_Output := Min_Valve;   (* Close valve during fault *)
    Flow_SP := Min_Flow_SP;
ELSE
    Pressure_Fault := FALSE;

    (* 2. Outer Loop: Pressure PID Control *)
    Pressure_Error := Pressure_SP - Pressure_PV;  (* Calculate pressure error *)
    Pressure_Integral := Pressure_Integral + (Pressure_Error * CycleTime);  (* Update integral *)
    Pressure_Derivative := (Pressure_Error - Pressure_Prev_Error) / CycleTime;  (* Derivative *)
    Pressure_Output := (Kp1 * Pressure_Error) + (Ki1 * Pressure_Integral) + (Kd1 * Pressure_Derivative);  (* PID output *)

    (* Clamp Pressure Output (Flow Setpoint) *)
    Flow_SP := Pressure_Output;
    IF Flow_SP > Max_Flow_SP THEN
        Flow_SP := Max_Flow_SP;  (* Limit flow setpoint *)
        Pressure_Integral := Pressure_Integral - (Pressure_Error * CycleTime);  (* Anti-windup *)
    ELSIF Flow_SP < Min_Flow_SP THEN
        Flow_SP := Min_Flow_SP;
        Pressure_Integral := Pressure_Integral - (Pressure_Error * CycleTime);  (* Anti-windup *)
    END_IF;

    (* Update previous error for pressure loop *)
    Pressure_Prev_Error := Pressure_Error;

    (* 3. Inner Loop: Flow PID Control *)
    Flow_Error := Flow_SP - Flow_PV;  (* Calculate flow error *)
    Flow_Integral := Flow_Integral + (Flow_Error * CycleTime);  (* Update integral *)
    Flow_Derivative := (Flow_Error - Flow_Prev_Error) / CycleTime;  (* Derivative *)
    Flow_Output := (Kp2 * Flow_Error) + (Ki2 * Flow_Integral) + (Kd2 * Flow_Derivative);  (* PID output *)

    (* Clamp Flow Output (Valve Position) *)
    Valve_Output := Flow_Output;
    IF Valve_Output > Max_Valve THEN
        Valve_Output := Max_Valve;  (* Limit valve opening *)
        Flow_Integral := Flow_Integral - (Flow_Error * CycleTime);  (* Anti-windup *)
    ELSIF Valve_Output < Min_Valve THEN
        Valve_Output := Min_Valve;
        Flow_Integral := Flow_Integral - (Flow_Error * CycleTime);  (* Anti-windup *)
    END_IF;

    (* Update previous error for flow loop *)
    Flow_Prev_Error := Flow_Error;
END_IF;

(* Notes:
   - Cascade Control:
     - Outer loop (pressure) regulates Pressure_PV to Pressure_SP (10.0 bar)
     - Inner loop (flow) adjusts Flow_PV to Flow_SP from Pressure_Output
   - PID Tuning:
     - Pressure: Kp1=1.5, Ki1=0.3, Kd1=0.1 for stable control
     - Flow: Kp2=2.0, Ki2=0.5, Kd2=0.2 for fast disturbance rejection
   - Safety:
     - Pressure_Fault triggers if Pressure_PV outside 8.0–12.0 bar
     - Valve_Output clamped (0.0–100.0%), Flow_SP clamped (0.0–100.0 L/min)
     - Anti-windup on Integral terms prevents overshoot
   - Timing:
     - CycleTime (0.1 s) used as REAL for numeric calculations
   - Physical Integration:
     - Pressure_PV: Analog input from pressure transducer
     - Flow_PV: Analog input from flow sensor
     - Valve_Output: Analog output to valve (e.g., 4–20 mA)
   - Scalability:
     - Adjust Kp, Ki, Kd for different dynamics
     - Add feedforward for known disturbances
   - Maintenance:
     - Add HMI for Pressure_PV, Flow_PV, Flow_SP, Valve_Output, Pressure_Fault
     - Log faults and PID terms for diagnostics
*)
END_PROGRAM
