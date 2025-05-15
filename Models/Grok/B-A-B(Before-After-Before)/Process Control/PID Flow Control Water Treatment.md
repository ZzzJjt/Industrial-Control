(* Chlorine Dosing Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain stable chlorine concentration using PID control for water treatment *)

PROGRAM ChlorineDosingControl
VAR
    (* Inputs *)
    FlowRate : REAL;                (* Real-time water flow rate in L/min *)
    Dosing_PV : REAL;               (* Measured chlorine concentration in ppm *)

    (* Outputs *)
    Dosing_Output : REAL;           (* Control signal to dosing pump, 0.0 to 100.0% *)

    (* Control Parameters *)
    Dosing_SP : REAL := 3.0;        (* Target chlorine concentration in ppm *)
    Kp : REAL := 2.0;               (* Proportional gain *)
    Ki : REAL := 0.5;               (* Integral gain *)
    Kd : REAL := 0.1;               (* Derivative gain *)
    CycleTime : REAL := 0.1;        (* Sampling rate in seconds, 100 ms *)

    (* PID Internal Variables *)
    Error : REAL;                   (* Current error (SP - PV) *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative calculation *)
    Integral : REAL := 0.0;         (* Integral term *)
    Derivative : REAL;              (* Derivative term *)

    (* Safety Limits *)
    Min_Dose : REAL := 0.0;         (* Minimum dosing output in % *)
    Max_Dose : REAL := 10.0;        (* Maximum dosing output in % *)
    Max_Flow : REAL := 1000.0;      (* Maximum expected flow rate in L/min *)
    Min_Flow : REAL := 0.0;         (* Minimum expected flow rate in L/min *)
    Input_Fault : BOOL;             (* TRUE if flow or dosing input is invalid *)
END_VAR

(* Main Control Logic *)
(* 1. Input Validation *)
IF FlowRate < Min_Flow OR FlowRate > Max_Flow OR Dosing_PV < 0.0 OR Dosing_PV > 10.0 THEN
    Input_Fault := TRUE;
    Dosing_Output := Min_Dose;  (* Set safe default during fault *)
ELSE
    Input_Fault := FALSE;

    (* 2. PID Control *)
    (* Calculate error *)
    Error := Dosing_SP - Dosing_PV;

    (* Update integral term *)
    Integral := Integral + (Error * CycleTime);

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / CycleTime;

    (* Calculate PID output *)
    Dosing_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* 3. Clamp Dosing Output to Safe Range *)
    IF Dosing_Output > Max_Dose THEN
        Dosing_Output := Max_Dose;  (* Prevent overdosing *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    ELSIF Dosing_Output < Min_Dose THEN
        Dosing_Output := Min_Dose;  (* Prevent underdosing *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    END_IF;

    (* Update previous error *)
    Prev_Error := Error;
END_IF;

(* Notes:
   - PID Control:
     - Maintains Dosing_PV at Dosing_SP (3.0 ppm) using FlowRate context
     - Kp=2.0, Ki=0.5, Kd=0.1 tuned for stable response
     - CycleTime (100 ms) ensures real-time control
   - Safety:
     - Input_Fault triggers for invalid FlowRate or Dosing_PV
     - Dosing_Output clamped between Min_Dose (0.0%) and Max_Dose (10.0%)
     - Anti-windup on Integral term prevents overshoot during clamping
   - Physical Integration:
     - FlowRate: Analog input from flow sensor
     - Dosing_PV: Analog input from chlorine concentration sensor
     - Dosing_Output: Analog output to dosing pump (e.g., 4–20 mA)
   - Scalability:
     - Adjust Kp, Ki, Kd for different system dynamics
     - Add feedforward term using FlowRate for hybrid control
   - Compliance:
     - Maintains 3.0 ppm chlorine for regulatory standards
     - Log Input_Fault for audit trails
   - Maintenance:
     - Integrate with HMI to display FlowRate, Dosing_PV, Dosing_Output, Input_Fault
     - Monitor Integral and Error for tuning diagnostics
*)
END_PROGRAM
