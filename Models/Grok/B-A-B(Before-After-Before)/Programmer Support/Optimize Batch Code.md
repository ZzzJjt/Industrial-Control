(* Optimized Polyethylene Batch Control in IEC 61131-3 Structured Text *)
(* Purpose: Manages a multi-step polyethylene batch process with modular state machine *)

PROGRAM PolyethyleneBatchControl
VAR
    (* State Machine *)
    State : INT := 0;               (* Current process state: 0=Idle, 1=RawMatPrep, ..., 7=Packaging *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Idle, waiting to start *)
        STATE_RAW_MAT_PREP : INT := 1; (* Raw material preparation *)
        STATE_POLYMERIZATION : INT := 2; (* Polymerization *)
        STATE_QUENCHING : INT := 3; (* Quenching *)
        STATE_DRYING : INT := 4;    (* Drying *)
        STATE_PELLETIZING : INT := 5; (* Pelletizing *)
        STATE_QUALITY_CONTROL : INT := 6; (* Quality control *)
        STATE_PACKAGING : INT := 7; (* Packaging and storage *)
    END_CONSTANT

    (* Inputs *)
    StartBatch : BOOL;              (* TRUE to initiate batch process *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Pressure_PV : REAL;             (* Measured pressure in bar *)

    (* Outputs *)
    Temp_SP : REAL;                 (* Temperature setpoint in °C *)
    Pressure_SP : REAL;             (* Pressure setpoint in bar *)
    Batch_Complete : BOOL;          (* TRUE when batch process is complete *)

    (* Timing *)
    StepTimer : TON;                (* Timer for step duration *)
    StepStartTime : TIME;           (* Start time of current step *)

    (* Process Parameters *)
    RawMatPrep_Temp : REAL := 70.0; (* Temperature setpoint for raw material prep *)
    RawMatPrep_Pressure : REAL := 1.0; (* Pressure setpoint for raw material prep *)
    Polymerization_Temp : REAL := 150.0; (* Temperature setpoint for polymerization *)
    Polymerization_Pressure : REAL := 30.0; (* Pressure setpoint for polymerization *)
    Quenching_Temp : REAL := 25.0;  (* Temperature setpoint for quenching *)
    Quenching_Pressure : REAL := 5.0; (* Pressure setpoint for quenching *)
    Drying_Temp : REAL := 80.0;     (* Temperature setpoint for drying *)
    Pelletizing_Temp : REAL := 150.0; (* Temperature setpoint for pelletizing *)
    QualityControl_Temp : REAL := 25.0; (* Temperature setpoint for quality control *)
    Packaging_Temp : REAL := 20.0;  (* Temperature setpoint for packaging *)
    Common_Pressure : REAL := 5.0;  (* Common pressure for drying, pelletizing, quality control, packaging *)

    (* Step Durations *)
    RawMatPrep_Duration : TIME := T#5s; (* Duration for raw material prep *)
    Polymerization_Duration : TIME := T#30m; (* Duration for polymerization *)
    Quenching_Duration : TIME := T#15m; (* Duration for quenching *)
    Drying_Duration : TIME := T#1h; (* Duration for drying *)
    Pelletizing_Duration : TIME := T#1h30m; (* Duration for pelletizing *)
    QualityControl_Duration : TIME := T#2h; (* Duration for quality control *)
    Packaging_Duration : TIME := T#3h; (* Duration for packaging *)
END_VAR

(* Method: Set Temperature and Pressure Setpoints *)
METHOD PRIVATE SetTemperatureAndPressure : BOOL
VAR_INPUT
    Temp : REAL;                    (* Desired temperature setpoint *)
    Pressure : REAL;                (* Desired pressure setpoint *)
END_VAR
Temp_SP := Temp;
Pressure_SP := Pressure;
(* Simulate setting conditions, e.g., via analog outputs to controllers *)
SetTemperatureAndPressure := TRUE;
END_METHOD

(* Method: Update Process Conditions Based on State *)
METHOD PRIVATE UpdateConditions : BOOL
CASE State OF
    STATE_RAW_MAT_PREP:
        SetTemperatureAndPressure(RawMatPrep_Temp, RawMatPrep_Pressure);
    STATE_POLYMERIZATION:
        SetTemperatureAndPressure(Polymerization_Temp, Polymerization_Pressure);
    STATE_QUENCHING:
        SetTemperatureAndPressure(Quenching_Temp, Quenching_Pressure);
    STATE_DRYING:
        SetTemperatureAndPressure(Drying_Temp, Common_Pressure);
    STATE_PELLETIZING:
        SetTemperatureAndPressure(Pelletizing_Temp, Common_Pressure);
    STATE_QUALITY_CONTROL:
        SetTemperatureAndPressure(QualityControl_Temp, Common_Pressure);
    STATE_PACKAGING:
        SetTemperatureAndPressure(Packaging_Temp, Common_Pressure);
    ELSE
        SetTemperatureAndPressure(0.0, 0.0); (* Safe default in Idle *)
END_CASE;
UpdateConditions := TRUE;
END_METHOD

(* Main Cyclic Logic *)
IF StartBatch AND State = STATE_IDLE THEN
    State := STATE_RAW_MAT_PREP;     (* Start batch process *)
    StepStartTime := CURRENT_TIME;   (* Record start time *)
    Batch_Complete := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        StepTimer(IN := FALSE);     (* Timer off in Idle *)
        Batch_Complete := FALSE;
        UpdateConditions();         (* Set safe defaults *)

    STATE_RAW_MAT_PREP:
        StepTimer(IN := TRUE, PT := RawMatPrep_Duration);
        IF StepTimer.Q THEN
            State := STATE_POLYMERIZATION;
            StepTimer(IN := FALSE);
            StepStartTime := CURRENT_TIME;
        END_IF;
        UpdateConditions();

    STATE_POLYMERIZATION:
        StepTimer(IN := TRUE, PT := Polymerization_Duration);
        IF StepTimer.Q THEN
            State := STATE_QUENCHING;
            StepTimer(IN := FALSE);
            StepStartTime := CURRENT_TIME;
        END_IF;
        UpdateConditions();

    STATE_QUENCHING:
        StepTimer(IN := TRUE, PT := Quenching_Duration);
        IF StepTimer.Q THEN
            State := STATE_DRYING;
            StepTimer(IN := FALSE);
            StepStartTime := CURRENT_TIME;
        END_IF;
        UpdateConditions();

    STATE_DRYING:
        StepTimer(IN := TRUE, PT := Drying_Duration);
        IF StepTimer.Q THEN
            State := STATE_PELLETIZING;
            StepTimer(IN := FALSE);
            StepStartTime := CURRENT_TIME;
        END_IF;
        UpdateConditions();

    STATE_PELLETIZING:
        StepTimer(IN := TRUE, PT := Pelletizing_Duration);
        IF StepTimer.Q THEN
            State := STATE_QUALITY_CONTROL;
            StepTimer(IN := FALSE);
            StepStartTime := CURRENT_TIME;
        END_IF;
        UpdateConditions();

    STATE_QUALITY_CONTROL:
        StepTimer(IN := TRUE, PT := QualityControl_Duration);
        IF StepTimer.Q THEN
            State := STATE_PACKAGING;
            StepTimer(IN := FALSE);
            StepStartTime := CURRENT_TIME;
        END_IF;
        UpdateConditions();

    STATE_PACKAGING:
        StepTimer(IN := TRUE, PT := Packaging_Duration);
        IF StepTimer.Q THEN
            State := STATE_IDLE;
            StepTimer(IN := FALSE);
            Batch_Complete := TRUE;
        END_IF;
        UpdateConditions();

ELSE
    State := STATE_IDLE;            (* Fallback to safe state *)
    StepTimer(IN := FALSE);
    Batch_Complete := FALSE;
    UpdateConditions();
END_CASE;

(* Notes:
   - Fixes Applied:
     - Removed LOOP constructs, using PLC cyclic execution for non-blocking behavior
     - Modularized state logic with clear state machine and UpdateConditions method
     - Standardized timer handling: StepTimer.IN set TRUE at state entry, FALSE at exit
     - Improved readability with named constants and structured methods
   - Safety:
     - Fallback to Idle state for undefined states
     - Safe temperature/pressure defaults (0.0) in Idle
     - Batch_Complete signals completion only after full cycle
   - Timing:
     - StepTimer manages step durations (5s to 3h)
     - StepStartTime tracks step entry via CURRENT_TIME for diagnostics
   - Physical Integration:
     - Inputs: StartBatch (operator button), Temp_PV/Pressure_PV (sensors)
     - Outputs: Temp_SP/Pressure_SP (controller setpoints), Batch_Complete (HMI signal)
   - Scalability:
     - Add new states by extending CASE and constants
     - Adjust durations or setpoints for different processes
   - Maintenance:
     - Add HMI to display State, StepTimer.ET, Temp_PV, Pressure_PV
     - Log StepStartTime for batch tracking
*)
END_PROGRAM
