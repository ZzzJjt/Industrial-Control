(* Chemical Reactor Pressure Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain stable reactor pressure using PID control for safety and efficiency *)

PROGRAM ReactorPressureControl
VAR
    (* Inputs *)
    Pressure_PV : REAL;             (* Measured reactor pressure in bar *)
    Pressure_SP : REAL := 5.0;      (* Setpoint for reactor pressure in bar *)

    (* Outputs *)
    Valve_Output : REAL;            (* Control signal to pressure control valve, 0.0 to 100.0% *)

    (* PID Parameters *)
    Kp : REAL := 2.0;               (* Proportional gain *)
    Ki : REAL := 0.8;               (* Integral gain *)
    Kd : REAL := 0.3;               (* Derivative gain *)
    CycleTime : REAL := 0.1;        (* Sampling rate in seconds, 100 ms *)

    (* PID Internal Variables *)
    Error : REAL;                   (* Current error (SP - PV) *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative calculation *)
    Integral : REAL := 0.0;         (* Integral term *)
    Derivative : REAL;              (* Derivative term *)

    (* Safety Limits *)
    Valve_Min : REAL := 0.0;        (* Minimum valve opening in % *)
    Valve_Max : REAL := 100.0;      (* Maximum valve opening in % *)
    Max_Pressure : REAL := 7.0;     (* Maximum allowable pressure in bar *)
    Min_Pressure : REAL := 3.0;     (* Minimum allowable pressure in bar *)
    Pressure_Fault : BOOL;          (* TRUE if pressure input is invalid *)
END_VAR

(* Main Control Logic *)
(* 1. Input Validation *)
IF Pressure_PV < Min_Pressure OR Pressure_PV > Max_Pressure THEN
    Pressure_Fault := TRUE;
    Valve_Output := Valve_Min;  (* Close valve during fault for safety *)
ELSE
    Pressure_Fault := FALSE;

    (* 2. PID Control *)
    (* Calculate error *)
    Error := Pressure_SP - Pressure_PV;

    (* Update integral term *)
    Integral := Integral + (Error * CycleTime);

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / CycleTime;

    (* Calculate PID output *)
    Valve_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* 3. Clamp Valve Output to Safe Range *)
    IF Valve_Output > Valve_Max THEN
        Valve_Output := Valve_Max;  (* Prevent over-pressurization *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    ELSIF Valve_Output < Valve_Min THEN
        Valve_Output := Valve_Min;  (* Prevent under-pressurization *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    END_IF;

    (* Update previous error *)
    Prev_Error := Error;
END_IF;

(* Notes:
   - PID Control:
     - Regulates Pressure_PV to Pressure_SP (5.0 bar) by adjusting Valve_Output
     - Kp=2.0, Ki=0.8, Kd=0.3 tuned for stable and responsive control
     - CycleTime (100 ms) ensures real-time response
   - Safety:
     - Pressure_Fault triggers for invalid Pressure_PV (<3.0 or >7.0 bar)
     - Valve_Output clamped between Valve_Min (0.0%) and Valve_Max (100.0%)
     - Anti-windup on Integral term prevents overshoot during clamping
   - Physical Integration:
     - Pressure_PV: Analog input from pressure transducer
     - Valve_Output: Analog output to pressure control valve (e.g., 4–20 mA)
   - Scalability:
     - Adjust Kp, Ki, Kd for different reactor dynamics
     - Add feedforward term for known disturbances (e.g., reaction rate changes)
   - Maintenance:
     - Integrate with HMI to display Pressure_PV, Valve_Output, Pressure_Fault
     - Log Integral, Error, and faults for tuning and diagnostics
*)
END_PROGRAM
