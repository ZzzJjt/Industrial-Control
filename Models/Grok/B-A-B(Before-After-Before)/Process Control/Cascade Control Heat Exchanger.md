(* Heat Exchanger Cascade Control in IEC 61131-3 Structured Text *)
(* Purpose: Regulate outlet temperature using cascade control with inner flow loop *)

PROGRAM HeatExchangerControl
VAR
    (* Outer Loop Inputs (Temperature) *)
    Temp_SP : REAL := 85.0;         (* Setpoint for outlet temperature in °C *)
    Temp_PV : REAL;                 (* Measured outlet temperature in °C *)

    (* Inner Loop Inputs (Flow) *)
    Flow_PV : REAL;                 (* Measured flow rate of heating medium in L/min *)

    (* Control Outputs *)
    Flow_SP : REAL;                 (* Setpoint for flow rate, derived from outer loop *)
    ValveOutput : REAL;             (* Control signal to flow valve, 0.0 to 100.0% *)

    (* Internal Variables *)
    Temp_Error : REAL;              (* Error for temperature loop *)
    Temp_Output : REAL;             (* Output of temperature loop *)
    Flow_Error : REAL;              (* Error for flow loop *)
    Flow_Output : REAL;             (* Output of flow loop *)

    (* Tuning Parameters *)
    Kp_Outer : REAL := 1.0;         (* Proportional gain for outer (temperature) loop *)
    Ki_Outer : REAL := 0.1;         (* Integral gain for outer loop *)
    Kp_Inner : REAL := 2.0;         (* Proportional gain for inner (flow) loop *)
    Ki_Inner : REAL := 0.2;         (* Integral gain for inner loop *)
    Temp_Integral : REAL;           (* Integral term for outer loop *)
    Flow_Integral : REAL;           (* Integral term for inner loop *)
    CycleTime : REAL := 0.1;        (* Control loop cycle time in seconds, e.g., 100 ms *)

    (* Safety Limits *)
    MaxValveOutput : REAL := 100.0; (* Maximum valve opening in % *)
    MinValveOutput : REAL := 0.0;   (* Minimum valve opening in % *)
    MaxFlowSP : REAL := 100.0;      (* Maximum flow setpoint in L/min *)
    MinFlowSP : REAL := 0.0;        (* Minimum flow setpoint in L/min *)
END_VAR

(* Main Control Logic *)
(* 1. Outer Loop: Temperature Control *)
Temp_Error := Temp_SP - Temp_PV;  (* Calculate temperature error *)
Temp_Integral := Temp_Integral + (Temp_Error * CycleTime);  (* Update integral term *)
Temp_Output := (Kp_Outer * Temp_Error) + (Ki_Outer * Temp_Integral);  (* PI control *)

(* 2. Set Flow Setpoint for Inner Loop *)
Flow_SP := Temp_Output;  (* Map outer loop output to flow setpoint *)
IF Flow_SP > MaxFlowSP THEN
    Flow_SP := MaxFlowSP;  (* Limit flow setpoint *)
ELSIF Flow_SP < MinFlowSP THEN
    Flow_SP := MinFlowSP;
END_IF;

(* 3. Inner Loop: Flow Control *)
Flow_Error := Flow_SP - Flow_PV;  (* Calculate flow error *)
Flow_Integral := Flow_Integral + (Flow_Error * CycleTime);  (* Update integral term *)
Flow_Output := (Kp_Inner * Flow_Error) + (Ki_Inner * Flow_Integral);  (* PI control *)

(* 4. Valve Output *)
ValveOutput := Flow_Output;  (* Map flow output to valve control *)
IF ValveOutput > MaxValveOutput THEN
    ValveOutput := MaxValveOutput;  (* Limit valve output *)
ELSIF ValveOutput < MinValveOutput THEN
    ValveOutput := MinValveOutput;
END_IF;

(* Notes:
   - Cascade Control:
     - Outer loop (temperature) regulates Temp_PV to Temp_SP (85°C)
     - Inner loop (flow) adjusts Flow_PV to Flow_SP derived from Temp_Output
   - Control Strategy:
     - Uses simplified PI control for both loops (proportional + integral)
     - CycleTime (100 ms) ensures real-time response
   - Tuning:
     - Kp_Outer=1.0, Ki_Outer=0.1 for stable temperature control
     - Kp_Inner=2.0, Ki_Inner=0.2 for fast flow response
   - Safety:
     - Limits ValveOutput (0–100%) and Flow_SP (0–100 L/min) to prevent overdriving
     - Integral terms prevent windup implicitly via output limits
   - Physical Integration:
     - Temp_PV: Analog input from temperature sensor (e.g., thermocouple)
     - Flow_PV: Analog input from flow sensor (e.g., mass flow meter)
     - ValveOutput: Analog output to control valve (e.g., 4–20 mA)
   - Scalability:
     - Add derivative term (Kd) for full PID control
     - Adjust tuning parameters (Kp, Ki) for different systems
   - Maintenance:
     - Integrate with HMI to display Temp_PV, Flow_PV, Flow_SP, ValveOutput
     - Log errors or deviations for diagnostics
*)
END_PROGRAM
