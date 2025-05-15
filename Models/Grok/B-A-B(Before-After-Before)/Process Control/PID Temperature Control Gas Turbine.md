(* Gas Turbine Temperature Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain stable turbine temperature using PID control for performance and safety *)

PROGRAM GasTurbineTempControl
VAR
    (* Inputs *)
    Temp_PV : REAL;                 (* Measured turbine temperature in °C *)
    Temp_SP : REAL := 950.0;        (* Setpoint for turbine temperature in °C *)

    (* Outputs *)
    Valve_Position : REAL;          (* Control signal to inlet valve, 0.0 to 100.0% *)

    (* PID Parameters *)
    Kp : REAL := 3.0;               (* Proportional gain *)
    Ki : REAL := 0.7;               (* Integral gain *)
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
    Max_Temp : REAL := 1000.0;      (* Maximum allowable temperature in °C *)
    Min_Temp : REAL := 900.0;       (* Minimum allowable temperature in °C *)
    Temp_Fault : BOOL;              (* TRUE if temperature input is invalid *)
END_VAR

(* Main Control Logic *)
(* 1. Input Validation *)
IF Temp_PV < Min_Temp OR Temp_PV > Max_Temp THEN
    Temp_Fault := TRUE;
    Valve_Position := Valve_Min;  (* Close valve during fault for safety *)
ELSE
    Temp_Fault := FALSE;

    (* 2. PID Control *)
    (* Calculate error *)
    Error := Temp_SP - Temp_PV;

    (* Update integral term *)
    Integral := Integral + (Error * CycleTime);

    (* Calculate derivative term *)
    Derivative := (Error - Prev_Error) / CycleTime;

    (* Calculate PID output *)
    Valve_Position := (Kp * Error) + (Ki * Integral) + (Kd * Derivative);

    (* 3. Clamp Valve Output to Safe Range *)
    IF Valve_Position > Valve_Max THEN
        Valve_Position := Valve_Max;  (* Prevent overheating *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    ELSIF Valve_Position < Valve_Min THEN
        Valve_Position := Valve_Min;  (* Prevent undercooling *)
        Integral := Integral - (Error * CycleTime);  (* Anti-windup *)
    END_IF;

    (* Update previous error *)
    Prev_Error := Error;
END_IF;

(* Notes:
   - PID Control:
     - Regulates Temp_PV to Temp_SP (950.0°C) by adjusting Valve_Position
     - Kp=3.0, Ki=0.7, Kd=0.2 tuned for fast and stable response
     - CycleTime (100 ms) ensures real-time control
   - Safety:
     - Temp_Fault triggers for invalid Temp_PV (<900.0 or >1000.0°C)
     - Valve_Position clamped between Valve_Min (0.0%) and Valve_Max (100.0%)
     - Anti-windup on Integral term prevents overshoot during clamping
   - Physical Integration:
     - Temp_PV: Analog input from temperature sensor (e.g., thermocouple)
     - Valve_Position: Analog output to inlet valve (e.g., 4–20 mA)
   - Scalability:
     - Adjust Kp, Ki, Kd for different turbine dynamics
     - Add feedforward term for known disturbances (e.g., load changes)
   - Maintenance:
     - Integrate with HMI to display Temp_PV, Valve_Position, Temp_Fault
     - Log Integral, Error, and faults for tuning and diagnostics
*)
END_PROGRAM
