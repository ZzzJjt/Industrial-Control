(* Pneumatic System Control in IEC 61131-3 Structured Text *)
(* Purpose: Maintain stable air flow and pressure with safety checks on a 100 ms loop *)

PROGRAM PneumaticControl
VAR
    (* Inputs *)
    FlowInput : REAL;                  (* Current flow rate from sensor in SLPM *)
    PressureInput : REAL;              (* Current pressure from sensor in bar *)

    (* Outputs *)
    FlowValveOutput : BOOL;            (* Controls flow valve: TRUE = open *)
    PressureReliefValve : BOOL;        (* Controls pressure relief valve: TRUE = open *)

    (* Internal Variables *)
    FlowSetpoint : REAL := 50.0;       (* Target flow rate: 50 SLPM *)
    MinPressure : REAL := 5.5;         (* Minimum allowable pressure: 5.5 bar *)
    MaxPressure : REAL := 6.0;         (* Maximum allowable pressure: 6.0 bar *)
    FlowError : BOOL;                  (* TRUE if flow deviation exceeds threshold *)
    PressureError : BOOL;              (* TRUE if pressure is out of range *)
    ControlLoopTimer : TON;            (* 100 ms timer for control loop *)
    LastCycleTime : TIME;              (* Tracks last cycle for timing *)
END_VAR

(* Main Control Logic *)
(* 1. Enforce 100 ms Control Loop *)
ControlLoopTimer(IN := TRUE, PT := T#100ms);
IF ControlLoopTimer.Q THEN
    ControlLoopTimer(IN := FALSE);  (* Reset timer for next cycle *)

    (* 2. Flow Control Logic *)
    IF FlowInput < FlowSetpoint THEN
        FlowValveOutput := TRUE;    (* Open valve to increase flow *)
    ELSIF FlowInput >= FlowSetpoint THEN
        FlowValveOutput := FALSE;   (* Close or hold valve to maintain flow *)
    END_IF;

    (* 3. Pressure Monitoring Logic *)
    IF PressureInput < MinPressure OR PressureInput > MaxPressure THEN
        PressureError := TRUE;
        PressureReliefValve := TRUE;  (* Open relief valve for safety *)
        FlowValveOutput := FALSE;     (* Stop flow during pressure fault *)
    ELSE
        PressureError := FALSE;
        PressureReliefValve := FALSE; (* Keep relief valve closed *)
    END_IF;

    (* 4. Flow Fault Detection *)
    IF ABS(FlowInput - FlowSetpoint) > 5.0 THEN
        FlowError := TRUE;           (* Flag significant flow deviation *)
    ELSE
        FlowError := FALSE;
    END_IF;

END_IF;

(* Notes:
   - Control Loop:
     - Runs every 100 ms using ControlLoopTimer to ensure consistent updates
   - Flow Regulation:
     - Maintains 50 SLPM by toggling FlowValveOutput based on FlowInput
     - Simple on/off control; can be extended to PID for finer regulation
   - Pressure Safety:
     - Keeps pressure within 5.5–6.0 bar
     - Opens PressureReliefValve and stops flow if pressure is out of range
   - Fault Handling:
     - FlowError flags deviations > 5 SLPM for diagnostics
     - PressureError flags out-of-range pressure for safety actions
   - Physical Integration:
     - FlowInput: Analog input from flow sensor (e.g., mass flow meter)
     - PressureInput: Analog input from pressure transducer
     - FlowValveOutput: Digital output to solenoid valve
     - PressureReliefValve: Digital output to relief valve
   - Scalability:
     - Add PID control for FlowValveOutput
     - Extend fault thresholds or add more sensors
   - Maintenance:
     - Integrate with HMI to display FlowInput, PressureInput, FlowError, PressureError
     - Log errors for diagnostics
*)
END_PROGRAM
