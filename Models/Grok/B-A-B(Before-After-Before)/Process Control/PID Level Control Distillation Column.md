(* Distillation Column Liquid Level Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain stable liquid level using PID control for optimal column performance *)

PROGRAM DistillationLevelControl
VAR
    (* Inputs *)
    Level_PV : REAL;                (* Measured liquid level in % *)
    Level_SP : REAL := 60.0;        (* Setpoint for liquid level in % *)

    (* Outputs *)
    Valve_Position : REAL;          (* Control signal to inlet valve, 0.0 to 100.0% *)

    (* PID Parameters *)
    Kp : REAL := 1.5;               (* Proportional gain *)
    Ki : REAL := 0.4;               (* Integral gain *)
    Kd : REAL := 0.2;               (* Derivative gain *)
    CycleTime : REAL := 0.1;        (* Sampling rate in seconds, 100 ms *)

    (* PID Internal Variables *)
    Error : REAL;                   (* Current error (SP - PV) *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative calculation *)
    Integral : REAL := 0.0;         (* Integral term *)
    Derivative : REAL;              (* Derivative term *)

    (* Safety Limits *)
    Valve_Min : REAL := 0.0;        (* Minimum valve opening in % *)
    Valve_Max : REAL := 100.0;      (* Maximum valve opening in % *)
    Max_Level : REAL := 100.0;      (* Maximum allowable level in % *)
    Min_Level : REAL := 0.0;        (* Minimum allowable level in % *)
    Level_Fault : BOOL;             (* TRUE if level input is invalid *)
END_VAR

(* Main Control Logic *)
(* 1. Input Validation *)
IF Level_PV < Min_Level OR Level_PV > Max_Level THEN
    Level_Fault := TRUE;
    Valve_Position := Valve_Min;  (* Close valve during fault for safety *)
ELSE
    Level_Fault := FALSE;

    (* 2. PID Control *)
    (* Calculate error *)
    Error := Level_SP - Level_PV;

    (* Update integral term *)
    Integral := Integral + (Error * CycleTime);

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / CycleTime;

    (* Calculate PID output *)
    Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* 3. Clamp Valve Position to Safe Range *)
    IF Valve_Position > Valve_Max THEN
        Valve_Position := Valve_Max;  (* Prevent overfeeding *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    ELSIF Valve_Position < Valve_Min THEN
        Valve_Position := Valve_Min;  (* Prevent underfeeding *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    END_IF;

    (* Update previous error *)
    Prev_Error := Error;
END_IF;

(* Notes:
   - PID Control:
     - Regulates Level_PV to Level_SP (60.0%) by adjusting Valve_Position
     - Kp=1.5, Ki=0.4, Kd=0.2 tuned for stable and responsive control
     - CycleTime (100 ms) ensures real-time response
   - Safety:
     - Level_Fault triggers for invalid Level_PV (<0.0 or >100.0%)
     - Valve_Position clamped between Valve_Min (0.0%) and Valve_Max (100.0%)
     - Anti-windup on Integral term prevents overshoot during clamping
   - Physical Integration:
     - Level_PV: Analog input from level sensor (e.g., ultrasonic or radar)
     - Valve_Position: Analog output to inlet valve (e.g., 4–20 mA)
   - Scalability:
     - Adjust Kp, Ki, Kd for different column dynamics
     - Add feedforward term for known disturbances (e.g., reflux rate)
   - Maintenance:
     - Integrate with HMI to display Level_PV, Valve_Position, Level_Fault
     - Log Integral, Error, and faults for tuning and diagnostics
*)
END_PROGRAM
