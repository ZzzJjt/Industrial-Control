(* IEC 61131-3 Structured Text: ISA-88 Urea Reaction Control *)
(* Purpose: Controls Reaction stage for urea fertilizer production *)

(* Function Block: Start Heating *)
FUNCTION_BLOCK StartHeating
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    Complete : BOOL;                (* TRUE when temperature reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Heating, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Heating *)
            HeaterOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Raises reactor temperature to target for reaction *)
END_FUNCTION_BLOCK

(* Function Block: Hold Pressure *)
FUNCTION_BLOCK HoldPressure
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
    Pressure_SP : REAL;             (* Target pressure in bar *)
    Pressure_Tol : REAL;            (* Pressure tolerance in bar *)
    Pressure_PV : REAL;             (* Measured pressure in bar *)
    Duration : TIME;                (* Reaction duration *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    CompressorOn : BOOL;            (* TRUE to activate compressor *)
    AgitatorOn : BOOL;              (* TRUE to activate agitator *)
    Complete : BOOL;                (* TRUE when duration complete *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Pressurizing, 2=Holding, 3=Complete *)
    HoldTimer : TON;                (* Timer for reaction duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    CompressorOn := FALSE;
    AgitatorOn := FALSE;
    Complete := FALSE;
    HoldTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            CompressorOn := FALSE;
            AgitatorOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Pressurizing *)
            CompressorOn := ABS(Pressure_PV - Pressure_SP) > Pressure_Tol;
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol;
            AgitatorOn := TRUE;
            IF ABS(Pressure_PV - Pressure_SP) <= Pressure_Tol AND 
               ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
                HoldTimer(IN := TRUE, PT := Duration);
            END_IF;
        2: (* Holding *)
            HeaterOn := ABS(Temp_PV - Temp_SP) > Temp_Tol;
            CompressorOn := ABS(Pressure_PV - Pressure_SP) > Pressure_Tol;
            AgitatorOn := TRUE;
            IF HoldTimer.Q THEN
                State := 3;
            END_IF;
        3: (* Complete *)
            HeaterOn := FALSE;
            CompressorOn := FALSE;
            AgitatorOn := FALSE;
            Complete := TRUE;
            HoldTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Maintains reaction conditions for urea formation *)
END_FUNCTION_BLOCK

(* Function Block: Start Cooling *)
FUNCTION_BLOCK StartCooling
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_SP : REAL;                 (* Target temperature in °C *)
    Temp_Tol : REAL;                (* Temperature tolerance in °C *)
    Temp_PV : REAL;                 (* Measured temperature in °C *)
END_VAR
VAR_OUTPUT
    CoolerOn : BOOL;                (* TRUE to activate cooler *)
    Complete : BOOL;                (* TRUE when temperature reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Cooling, 2=Complete *)
END_VAR
IF NOT Enable THEN
    State := 0;
    CoolerOn := FALSE;
    Complete := FALSE;
ELSE
    CASE State OF
        0: (* Idle *)
            CoolerOn := FALSE;
            Complete := FALSE;
            State := 1;
        1: (* Cooling *)
            CoolerOn := TRUE;
            IF ABS(Temp_PV - Temp_SP) <= Temp_Tol THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            CoolerOn := FALSE;
            Complete := TRUE;
    END_CASE;
END_IF;
(* Lowers reactor temperature post-reaction *)
END_FUNCTION_BLOCK

(* Main Program: Urea Reaction Control *)
PROGRAM UreaReactionControl
VAR
    (* Inputs *)
    StartBatch : BOOL;              (* TRUE to start batch *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    ReactorTemp_PV : REAL;          (* Reactor temperature in °C *)
    ReactorPressure_PV : REAL;      (* Reactor pressure in bar *)
    
    (* Outputs *)
    ReactorHeaterOn : BOOL;         (* TRUE to activate heater *)
    ReactorCompressorOn : BOOL;     (* TRUE to activate compressor *)
    ReactorAgitatorOn : BOOL;       (* TRUE to activate agitator *)
    ReactorCoolerOn : BOOL;         (* TRUE to activate cooler *)
    StageComplete : BOOL;           (* TRUE when reaction stage complete *)
    
    (* State Machine *)
    State : INT := 0;               (* 0=Idle, 1=StartHeating, 2=HoldPressure, 3=StartCooling, 4=Complete *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_HEATING : INT := 1;   (* Heating to reaction temp *)
        STATE_HOLD : INT := 2;      (* Holding reaction conditions *)
        STATE_COOLING : INT := 3;   (* Cooling post-reaction *)
        STATE_COMPLETE : INT := 4;  (* Reaction stage complete *)
    END_CONSTANT
    
    (* Recipe Parameters *)
    ReactionTemp_SP : REAL := 180.0; (* Reaction temperature: 180°C *)
    ReactionTemp_Tol : REAL := 3.0;  (* Temp tolerance: ±3°C *)
    ReactionPressure_SP : REAL := 150.0; (* Reaction pressure: 150 bar *)
    ReactionPressure_Tol : REAL := 5.0;  (* Pressure tolerance: ±5 bar *)
    ReactionDuration : TIME := T#2h; (* Reaction duration: 2 hours *)
    CoolingTemp_SP : REAL := 80.0;   (* Cooling temperature: 80°C *)
    CoolingTemp_Tol : REAL := 3.0;   (* Cooling tolerance: ±3°C *)
    
    (* Operation Instances *)
    HeatOp : StartHeating;          (* Heating operation *)
    HoldOp : HoldPressure;          (* Hold pressure operation *)
    CoolOp : StartCooling;          (* Cooling operation *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    ReactorHeaterOn := FALSE;
    ReactorCompressorOn := FALSE;
    ReactorAgitatorOn := FALSE;
    ReactorCoolerOn := FALSE;
    StageComplete := FALSE;
    HeatOp.Enable := FALSE;
    HoldOp.Enable := FALSE;
    CoolOp.Enable := FALSE;
ELSIF StartBatch AND State = STATE_IDLE THEN
    State := STATE_HEATING;
    StageComplete := FALSE;
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        ReactorHeaterOn := FALSE;
        ReactorCompressorOn := FALSE;
        ReactorAgitatorOn := FALSE;
        ReactorCoolerOn := FALSE;
        StageComplete := FALSE;
        HeatOp.Enable := FALSE;
        HoldOp.Enable := FALSE;
        CoolOp.Enable := FALSE;
        (* Idle state: All equipment off, ready for new batch *)

    STATE_HEATING:
        HeatOp(Enable := TRUE, Temp_SP := ReactionTemp_SP, Temp_Tol := ReactionTemp_Tol, Temp_PV := ReactorTemp_PV);
        ReactorHeaterOn := HeatOp.HeaterOn;
        IF HeatOp.Complete THEN
            State := STATE_HOLD;
            HeatOp.Enable := FALSE;
        END_IF;
        (* Heats reactor to 180°C for urea synthesis *)

    STATE_HOLD:
        HoldOp(Enable := TRUE, Temp_SP := ReactionTemp_SP, Temp_Tol := ReactionTemp_Tol, Temp_PV := ReactorTemp_PV, 
               Pressure_SP := ReactionPressure_SP, Pressure_Tol := ReactionPressure_Tol, Pressure_PV := ReactorPressure_PV, 
               Duration := ReactionDuration);
        ReactorHeaterOn := HoldOp.HeaterOn;
        ReactorCompressorOn := HoldOp.CompressorOn;
        ReactorAgitatorOn := HoldOp.AgitatorOn;
        IF HoldOp.Complete THEN
            State := STATE_COOLING;
            HoldOp.Enable := FALSE;
        END_IF;
        (* Maintains 180°C, 150 bar for 2 hours with agitation *)

    STATE_COOLING:
        CoolOp(Enable := TRUE, Temp_SP := CoolingTemp_SP, Temp_Tol := CoolingTemp_Tol, Temp_PV := ReactorTemp_PV);
        ReactorCoolerOn := CoolOp.CoolerOn;
        IF CoolOp.Complete THEN
            State := STATE_COMPLETE;
            CoolOp.Enable := FALSE;
        END_IF;
        (* Cools reactor to 80°C for safe transfer *)

    STATE_COMPLETE:
        ReactorHeaterOn := FALSE;
        ReactorCompressorOn := FALSE;
        ReactorAgitatorOn := FALSE;
        ReactorCoolerOn := FALSE;
        StageComplete := TRUE;
        IF NOT StartBatch THEN
            State := STATE_IDLE;
        END_IF;
        (* Reaction stage complete; reset for next stage *)

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Unit Procedure: Reaction stage with operations StartHeating, HoldPressure, StartCooling.
     - Operations: Implemented as reusable function blocks.
     - Recipe Parameters: Temp_SP, Pressure_SP, Duration for flexibility.
     - State Model: Idle, Running, Complete per ISA-88.
   - Reaction Stage:
     - StartHeating: Heat to 180°C ± 3°C.
     - HoldPressure: Maintain 180°C, 150 bar ± 5 bar for 2 hours with agitation.
     - StartCooling: Cool to 80°C ± 3°C.
   - Inputs: Ammonia (200 kg), CO2 (300 kg) dosed in A.1 (assumed pre-handled).
   - Outputs: Aqueous urea solution for concentration.
   - Safety:
     - EmergencyStop halts operations, resets state.
     - Tolerances (3°C, 5 bar) ensure quality and safety.
   - Quality:
     - Precise temperature and pressure control ensures high urea yield.
     - Agitation prevents side reactions.
   - Physical Integration:
     - Inputs: StartBatch (operator), Temp_PV, Pressure_PV (sensors), EmergencyStop.
     - Outputs: HeaterOn, CompressorOn, AgitatorOn, CoolerOn (relays).
   - Scalability:
     - Adjust parameters (Temp_SP, Duration) for larger reactors.
     - Add operations (e.g., dosing) as function blocks.
   - Maintenance:
     - Modular blocks simplify debugging.
     - Monitor Temp_PV, Pressure_PV, HoldOp.HoldTimer.ET on HMI.
*)
END_PROGRAM
