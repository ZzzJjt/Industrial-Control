(* Oil Refinery Vessel Cascade Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain stable vessel pressure using cascade control with inner flow loop *)

PROGRAM RefineryVesselControl
VAR
    (* Outer Loop Inputs (Pressure) *)
    Pressure_SP : REAL := 12.0;     (* Setpoint for vessel pressure in bar *)
    Pressure_PV : REAL;             (* Measured vessel pressure in bar *)

    (* Inner Loop Inputs (Flow) *)
    Flow_PV : REAL;                 (* Measured oil inflow rate in L/min *)

    (* Control Outputs *)
    Flow_SP : REAL;                 (* Setpoint for oil flow, derived from outer loop *)
    ValveOutput : REAL;             (* Control signal to oil inflow valve, 0.0 to 100.0% *)

    (* Internal Variables *)
    Pressure_Error : REAL;          (* Error for pressure loop *)
    Pressure_Output : REAL;         (* Output of pressure loop *)
    Flow_Error : REAL;              (* Error for flow loop *)
    Flow_Output : REAL;             (* Output of flow loop *)
    Pressure_Integral : REAL;       (* Integral term for outer loop *)
    Flow_Integral : REAL;           (* Integral term for inner loop *)

    (* Tuning Parameters *)
    Kp_Outer : REAL := 1.2;         (* Proportional gain for outer (pressure) loop *)
    Ki_Outer : REAL := 0.15;        (* Integral gain for outer loop *)
    Kp_Inner : REAL := 2.5;         (* Proportional gain for inner (flow) loop *)
    Ki_Inner : REAL := 0.3;         (* Integral gain for inner loop *)
    CycleTime : REAL := 0.1;        (* Control loop cycle time in seconds, 100 ms *)

    (* Safety Limits *)
    MaxValveOutput : REAL := 100.0; (* Maximum valve opening in % *)
    MinValveOutput : REAL := 0.0;   (* Minimum valve opening in % *)
    MaxFlowSP : REAL := 150.0;      (* Maximum flow setpoint in L/min *)
    MinFlowSP : REAL := 0.0;        (* Minimum flow setpoint in L/min *)
    MaxPressure : REAL := 15.0;     (* Maximum allowable pressure in bar *)
    MinPressure : REAL := 9.0;      (* Minimum allowable pressure in bar *)
    PressureFault : BOOL;           (* TRUE if pressure out of safe range *)
END_VAR

(* Main Control Logic *)
(* 1. Safety Check: Pressure Range *)
IF Pressure_PV > MaxPressure OR Pressure_PV < MinPressure THEN
    PressureFault := TRUE;
    ValveOutput := MinValveOutput;  (* Close valve for safety *)
    Flow_SP := MinFlowSP;
ELSE
    PressureFault := FALSE;

    (* 2. Outer Loop: Pressure Control *)
    Pressure_Error := Pressure_SP - Pressure_PV;  (* Calculate pressure error *)
    Pressure_Integral := Pressure_Integral + (Pressure_Error * CycleTime);  (* Update integral *)
    Pressure_Output := (Kp_Outer * Pressure_Error) + (Ki_Outer * Pressure_Integral);  (* PI control *)

    (* 3. Set Flow Setpoint for Inner Loop *)
    Flow_SP := Pressure_Output;  (* Map outer loop output to flow setpoint *)
    IF Flow_SP > MaxFlowSP THEN
        Flow_SP := MaxFlowSP;   (* Limit flow setpoint *)
    ELSIF Flow_SP < MinFlowSP THEN
        Flow_SP := MinFlowSP;
    END_IF;

    (* 4. Inner Loop: Flow Control *)
    Flow_Error := Flow_SP - Flow_PV;  (* Calculate flow error *)
    Flow_Integral := Flow_Integral + (Flow_Error * CycleTime);  (* Update integral *)
    Flow_Output := (Kp_Inner * Flow_Error) + (Ki_Inner * Flow_Integral);  (* PI control *)

    (* 5. Valve Output *)
    ValveOutput := Flow_Output;  (* Map flow output to valve control *)
    IF ValveOutput > MaxValveOutput THEN
        ValveOutput := MaxValveOutput;  (* Limit valve output *)
    ELSIF ValveOutput < MinValveOutput THEN
        ValveOutput := MinValveOutput;
    END_IF;
END_IF;

(* Notes:
   - Cascade Control:
     - Outer loop regulates Pressure_PV to Pressure_SP (12.0 bar)
     - Inner loop adjusts Flow_PV to Flow_SP from outer loop output
   - Control Strategy:
     - PI control for both loops (proportional + integral)
     - CycleTime (100 ms) ensures real-time response
   - Tuning:
     - Kp_Outer=1.2, Ki_Outer=0.15 for stable pressure control
     - Kp_Inner=2.5, Ki_Inner=0.3 for fast flow response
   - Safety:
     - PressureFault triggers if Pressure_PV outside 9.0–15.0 bar
     - Valve closes (ValveOutput=0) during fault
     - Limits ValveOutput (0–100%) and Flow_SP (0–150 L/min)
   - Physical Integration:
     - Pressure_PV: Analog input from pressure transducer
     - Flow_PV: Analog input from flow sensor (e.g., mass flow meter)
     - ValveOutput: Analog output to control valve (e.g., 4–20 mA)
   - Scalability:
     - Add derivative term (Kd) for full PID control
     - Adjust tuning parameters or limits for different vessels
   - Maintenance:
     - Integrate with HMI to display Pressure_PV, Flow_PV, Flow_SP, ValveOutput, PressureFault
     - Log faults or deviations for diagnostics
*)
END_PROGRAM
