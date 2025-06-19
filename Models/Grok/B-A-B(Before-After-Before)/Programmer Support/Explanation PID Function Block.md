(* PID Control Example in IEC 61131-3 Structured Text *)
(* Purpose: Demonstrate OSCAT-like PID function block for reactor temperature control *)

PROGRAM PIDControlExample
VAR
    (* Inputs to PID Function Block *)
    PV : REAL;                      (* Process Variable: Measured temperature in °C *)
    SP : REAL := 85.0;              (* Setpoint: Desired temperature in °C *)
    Kp : REAL := 2.0;               (* Proportional gain *)
    Ki : REAL := 0.5;               (* Integral gain *)
    Kd : REAL := 0.1;               (* Derivative gain *)
    CycleTime : REAL := 0.1;        (* Sampling time in seconds, 100 ms *)

    (* Outputs from PID Function Block *)
    Control_Output : REAL;          (* Control signal to heating valve, 0.0 to 100.0% *)
    Error : REAL;                   (* Current error: SP - PV *)
    Output_Saturated : BOOL;        (* TRUE if output is at min/max limit *)

    (* Internal PID Variables *)
    Integral : REAL := 0.0;         (* Integral term *)
    Prev_Error : REAL := 0.0;       (* Previous error for derivative calculation *)
    Derivative : REAL;              (* Derivative term *)

    (* Safety Limits *)
    Valve_Min : REAL := 0.0;        (* Minimum valve opening in % *)
    Valve_Max : REAL := 100.0;      (* Maximum valve opening in % *)
    Max_Temp : REAL := 95.0;        (* Maximum allowable temperature in °C *)
    Min_Temp : REAL := 75.0;        (* Minimum allowable temperature in °C *)
    Temp_Fault : BOOL;              (* TRUE if temperature input is invalid *)
END_VAR

(* Simulated PID Function Block Logic *)
(* 1. Input Validation *)
IF PV < Min_Temp OR PV > Max_Temp THEN
    Temp_Fault := TRUE;
    Control_Output := Valve_Min;  (* Close valve during fault *)
ELSE
    Temp_Fault := FALSE;

    (* 2. Calculate Error *)
    Error := SP - PV;

    (* 3. PID Calculations *)
    (* Proportional Term *)
    (* Integral Term *)
    Integral := Integral + (Error * CycleTime);

    (* Derivative Term *)
    Derivative := (Error - Prev_Error) / CycleTime;

    (* PID Output *)
    Control_Output := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* 4. Clamp Output and Check Saturation *)
    Output_Saturated := FALSE;
    IF Control_Output > Valve_Max THEN
        Control_Output := Valve_Max;  (* Clamp to max *)
        Output_Saturated := TRUE;
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    ELSIF Control_Output < Valve_Min THEN
        Control_Output := Valve_Min;  (* Clamp to min *)
        Output_Saturated := TRUE;
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    END_IF;

    (* Update Previous Error *)
    Prev_Error := Error;
END_IF;

(* Notes:
   - PID Function Block Simulation:
     - Mimics OSCAT PID block behavior with PV, SP, Kp, Ki, Kd inputs
     - Outputs Control_Output (valve position), Error, and Output_Saturated
   - Application:
     - Controls reactor temperature to SP (85°C) by adjusting heating valve
     - Kp=2.0, Ki=0.5, Kd=0.1 tuned for stable and responsive control
   - Safety:
     - Temp_Fault triggers if PV outside 75.0–95.0°C
     - Control_Output clamped between Valve_Min (0.0%) and Valve_Max (100.0%)
     - Anti-windup on Integral prevents overshoot during saturation
   - Benefits:
     - Optimizes process by minimizing temperature deviations
     - Improves stability with tuned PID gains and derivative damping
     - Flexible for applications like pressure or flow control
     - Reduces manual intervention via automated error correction
   - Physical Integration:
     - PV: Analog input from temperature sensor (e.g., thermocouple)
     - Control_Output: Analog output to valve (e.g., 4–20 mA)
   - Scalability:
     - Adjust Kp, Ki, Kd for different processes
     - Add feedforward for known disturbances
   - Maintenance:
     - Add HMI to display PV, SP, Control_Output, Error, Temp_Fault
     - Log Output_Saturated and faults for diagnostics
*)
END_PROGRAM
