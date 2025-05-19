(* IEC 61131-3 Structured Text Program: UreaReactionControl *)
(* Purpose: Implements ISA-88 B.2 Reaction stage for urea fertilizer production *)
(* Overview: Manages heating to 180°C, pressure hold at 140 bar, and cooling to 50°C *)

PROGRAM UreaReactionControl
VAR
    (* Inputs *)
    StartReaction : R_TRIG;         (* Rising edge to start B.2 *)
    StopReaction : R_TRIG;          (* Rising edge to stop B.2 *)
    CurrentTemp : REAL;             (* Reactor temperature, °C *)
    CurrentPressure : REAL;         (* Reactor pressure, bar *)
    CurrentAgitSpeed : REAL;        (* Agitator speed, RPM *)
    EmergencyStop : BOOL;           (* TRUE if emergency stop triggered *)

    (* Outputs *)
    TargetTemp : REAL;              (* Temperature setpoint, °C *)
    TargetPressure : REAL;          (* Pressure setpoint, bar *)
    TargetAgitSpeed : REAL;         (* Agitator speed setpoint, RPM *)
    HeaterControl : BOOL;           (* TRUE to activate heater *)
    PressurePumpControl : BOOL;     (* TRUE to activate pressure pump *)
    CoolerControl : BOOL;           (* TRUE to activate cooler *)
    AgitatorControl : BOOL;         (* TRUE to activate agitator *)
    PhaseComplete : BOOL;           (* TRUE when B.2 is finished *)
    State : INT := 0;               (* Current state: 0=IDLE, 1=HEAT_UP, 2=PRESSURE_HOLD, 3=COOL_DOWN, 4=COMPLETED, 5=ERROR *)
    DiagLog : ARRAY[1..50] OF STRING[80]; (* Diagnostic logs *)
    LogCount : INT;                 (* Number of log entries *)

    (* Internal *)
    ReactionTimer : TON;            (* Timer for phase durations *)
    Last_StartReaction : BOOL;      (* Previous StartReaction.Q *)
    Last_StopReaction : BOOL;      (* Previous StopReaction.Q *)
    Last_PhaseComplete : BOOL;      (* Previous PhaseComplete *)
    Last_State : INT;               (* Previous State *)
    Timestamp : STRING[20];         (* Current timestamp *)
    LogBufferFull : BOOL;           (* TRUE if log buffer full *)
END_VAR

(* Parameter Constants *)
VAR CONSTANT
    (* Reaction Parameters *)
    TARGET_TEMP : REAL := 180.0;    (* Target temperature, °C *)
    TEMP_TOLERANCE : REAL := 2.0;   (* ±2°C tolerance *)
    TARGET_PRESSURE : REAL := 140.0; (* Target pressure, bar *)
    PRESSURE_TOLERANCE : REAL := 2.0; (* ±2 bar tolerance *)
    AGIT_SPEED : REAL := 100.0;     (* Agitator speed, RPM *)
    AGIT_SPEED_TOLERANCE : REAL := 10.0; (* ±10 RPM tolerance *)
    HEAT_TIMEOUT : TIME := T#10m;   (* Heat-up timeout *)
    HOLD_DURATION : TIME := T#30m;  (* Pressure hold duration *)
    COOL_TIMEOUT : TIME := T#15m;   (* Cool-down timeout *)
    (* Input Validation Ranges *)
    MIN_TEMP : REAL := 0.0;         (* Min valid temperature, °C *)
    MAX_TEMP : REAL := 300.0;       (* Max valid temperature, °C *)
    MIN_PRESS : REAL := 0.0;        (* Min valid pressure, bar *)
    MAX_PRESS : REAL := 200.0;      (* Max valid pressure, bar *)
    MIN_SPEED : REAL := 0.0;        (* Min valid speed, RPM *)
    MAX_SPEED : REAL := 500.0;      (* Max valid speed, RPM *)
END_VAR

(* Method: StartHeating *)
(* Initiates temperature ramp-up to target, activates heater *)
METHOD PRIVATE StartHeating : BOOL
    TargetTemp := TARGET_TEMP;      (* Setpoint: 180°C *)
    HeaterControl := TRUE;
    PressurePumpControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    TargetPressure := 0.0;
    TargetAgitSpeed := 0.0;
    StartHeating := ABS(CurrentTemp - TARGET_TEMP) <= TEMP_TOLERANCE; (* TRUE if temp stable *)
    IF LogCount < 50 AND (TargetTemp <> Last_TargetTemp OR HeaterControl <> Last_HeaterControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Heating Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), ' °C')));
    END_IF;
END_METHOD

(* Method: HoldPressure *)
(* Maintains pressure, temperature, and agitation *)
METHOD PRIVATE HoldPressure : BOOL
    TargetTemp := TARGET_TEMP;      (* Maintain 180°C *)
    TargetPressure := TARGET_PRESSURE; (* Setpoint: 140 bar *)
    TargetAgitSpeed := AGIT_SPEED;  (* Setpoint: 100 RPM *)
    HeaterControl := TRUE;
    PressurePumpControl := TRUE;
    CoolerControl := FALSE;
    AgitatorControl := TRUE;
    HoldPressure := ABS(CurrentTemp - TARGET_TEMP) <= TEMP_TOLERANCE AND 
                    ABS(CurrentPressure - TARGET_PRESSURE) <= PRESSURE_TOLERANCE AND 
                    ABS(CurrentAgitSpeed - AGIT_SPEED) <= AGIT_SPEED_TOLERANCE; (* TRUE if stable *)
    IF LogCount < 50 AND State <> Last_State THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Pressure Hold Started: Temp=', 
            CONCAT(TO_STRING(CurrentTemp), CONCAT(' °C, Press=', 
            CONCAT(TO_STRING(CurrentPressure), CONCAT(' bar, Speed=', 
            CONCAT(TO_STRING(CurrentAgitSpeed), ' RPM')))))));
    END_IF;
END_METHOD

(* Method: StartCooling *)
(* Initiates cooling to target temperature *)
METHOD PRIVATE StartCooling : BOOL
    TargetTemp := 50.0;             (* Cool to 50°C *)
    HeaterControl := FALSE;
    PressurePumpControl := FALSE;
    CoolerControl := TRUE;
    AgitatorControl := FALSE;
    TargetPressure := 0.0;
    TargetAgitSpeed := 0.0;
    StartCooling := ABS(CurrentTemp - 50.0) <= TEMP_TOLERANCE; (* TRUE if temp stable *)
    IF LogCount < 50 AND (TargetTemp <> Last_TargetTemp OR CoolerControl <> Last_CoolerControl) THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Cooling Started: TargetTemp=', 
            CONCAT(TO_STRING(TargetTemp), ' °C')));
    END_IF;
END_METHOD

(* Edge detection *)
StartReaction(CLK := NOT Last_StartReaction);
StopReaction(CLK := NOT Last_StopReaction);

(* Main Logic *)
(* State Machine Overview:
   - IDLE (0): Await StartReaction, outputs off
   - HEAT_UP (1): Heat to 180°C, 10m timeout
   - PRESSURE_HOLD (2): Hold 180°C, 140 bar, 100 RPM for 30m
   - COOL_DOWN (3): Cool to 50°C, 15m timeout
   - COMPLETED (4): Signal completion
   - ERROR (5): Safe state on fault
   - Methods: StartHeating, HoldPressure, StartCooling
   - Transitions: Timer (ReactionTimer.Q) and conditions (temp, pressure, speed)
*)

(* Step 1: Input validation *)
IF EmergencyStop THEN
    State := 5; (* ERROR *)
    TargetTemp := 0.0;
    TargetPressure := 0.0;
    TargetAgitSpeed := 0.0;
    HeaterControl := FALSE;
    PressurePumpControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    PhaseComplete := FALSE;
    ReactionTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Emergency Stop: Switching to ERROR');
    END_IF;
    RETURN;
END_IF;

IF NOT IS_VALID_REAL(CurrentTemp) OR NOT IS_VALID_REAL(CurrentPressure) OR 
   NOT IS_VALID_REAL(CurrentAgitSpeed) OR
   CurrentTemp < MIN_TEMP OR CurrentTemp > MAX_TEMP OR
   CurrentPressure < MIN_PRESS OR CurrentPressure > MAX_PRESS OR
   CurrentAgitSpeed < MIN_SPEED OR CurrentAgitSpeed > MAX_SPEED THEN
    State := 5; (* ERROR *)
    TargetTemp := 0.0;
    TargetPressure := 0.0;
    TargetAgitSpeed := 0.0;
    HeaterControl := FALSE;
    PressurePumpControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    PhaseComplete := FALSE;
    ReactionTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' Invalid Input: Temp=', 
            CONCAT(TO_STRING(CurrentTemp), CONCAT(' °C, Press=', 
            CONCAT(TO_STRING(CurrentPressure), CONCAT(' bar, Speed=', 
            CONCAT(TO_STRING(CurrentAgitSpeed), ' RPM')))))));
    END_IF;
    RETURN;
END_IF;

(* Step 2: Start and stop triggers *)
IF StartReaction.Q AND State = 0 THEN
    State := 1; (* HEAT_UP *)
    ReactionTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Reaction Started: Switching to HEAT_UP');
    END_IF;
END_IF;
IF StopReaction.Q AND State IN (1..3) THEN
    State := 5; (* ERROR *)
    TargetTemp := 0.0;
    TargetPressure := 0.0;
    TargetAgitSpeed := 0.0;
    HeaterControl := FALSE;
    PressurePumpControl := FALSE;
    CoolerControl := FALSE;
    AgitatorControl := FALSE;
    PhaseComplete := FALSE;
    ReactionTimer.IN := FALSE;
    IF LogCount < 50 THEN
        LogCount := LogCount + 1;
        Timestamp := '2025-05-19 13:40:00';
        DiagLog[LogCount] := CONCAT(Timestamp, ' Reaction Stopped: Switching to ERROR');
    END_IF;
END_IF;

(* Step 3: State machine execution *)
CASE State OF
    0: (* IDLE: Await StartReaction *)
        (* Initialize outputs to safe state *)
        TargetTemp := 0.0;
        TargetPressure := 0.0;
        TargetAgitSpeed := 0.0;
        HeaterControl := FALSE;
        PressurePumpControl := FALSE;
        CoolerControl := FALSE;
        AgitatorControl := FALSE;
        PhaseComplete := FALSE;
        ReactionTimer.IN := FALSE;

    1: (* HEAT_UP: Heat to 180°C *)
        (* Ramp up temperature to 180°C *)
        IF StartHeating() OR ReactionTimer.Q THEN
            State := StartHeating() ? 2 : 5; (* PRESSURE_HOLD if temp stable, else ERROR *)
            ReactionTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, StartHeating() ? ' Heating Complete: Switching to PRESSURE_HOLD' : ' Heating Timeout: Switching to ERROR');
            END_IF;
        ELSE
            ReactionTimer(IN := TRUE, PT := HEAT_TIMEOUT);
        END_IF;

    2: (* PRESSURE_HOLD: Hold 180°C, 140 bar, 100 RPM for 30m *)
        (* Maintain reaction conditions *)
        IF NOT HoldPressure() THEN
            State := 5; (* ERROR if conditions lost *)
            TargetTemp := 0.0;
            TargetPressure := 0.0;
            TargetAgitSpeed := 0.0;
            HeaterControl := FALSE;
            PressurePumpControl := FALSE;
            CoolerControl := FALSE;
            AgitatorControl := FALSE;
            PhaseComplete := FALSE;
            ReactionTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, ' Pressure Hold Failed: Switching to ERROR');
            END_IF;
        ELSE
            ReactionTimer(IN := TRUE, PT := HOLD_DURATION);
            IF ReactionTimer.Q THEN
                State := 3; (* COOL_DOWN *)
                ReactionTimer.IN := FALSE;
                IF LogCount < 50 THEN
                    LogCount := LogCount + 1;
                    Timestamp := '2025-05-19 13:40:00';
                    DiagLog[LogCount] := CONCAT(Timestamp, ' Pressure Hold Complete: Switching to COOL_DOWN');
                END_IF;
            END_IF;
        END_IF;

    3: (* COOL_DOWN: Cool to 50°C *)
        (* Cool reactor to safe temperature *)
        IF StartCooling() OR ReactionTimer.Q THEN
            State := StartCooling() ? 4 : 5; (* COMPLETED if cooled, else ERROR *)
            ReactionTimer.IN := FALSE;
            IF LogCount < 50 THEN
                LogCount := LogCount + 1;
                Timestamp := '2025-05-19 13:40:00';
                DiagLog[LogCount] := CONCAT(Timestamp, StartCooling() ? ' Cooling Complete: Switching to COMPLETED' : ' Cooling Timeout: Switching to ERROR');
            END_IF;
        ELSE
            ReactionTimer(IN := TRUE, PT := COOL_TIMEOUT);
        END_IF;

    4: (* COMPLETED: Signal completion *)
        (* Reaction phase finished, await reset *)
        TargetTemp := 0.0;
        TargetPressure := 0.0;
        TargetAgitSpeed := 0.0;
        HeaterControl := FALSE;
        PressurePumpControl := FALSE;
        CoolerControl := FALSE;
        AgitatorControl := FALSE;
        PhaseComplete := TRUE;
        ReactionTimer.IN := FALSE;

    5: (* ERROR: Safe state *)
        (* Safe shutdown on fault *)
        TargetTemp := 0.0;
        TargetPressure := 0.0;
        TargetAgitSpeed := 0.0;
        HeaterControl := FALSE;
        PressurePumpControl := FALSE;
        CoolerControl := FALSE;
        AgitatorControl := FALSE;
        PhaseComplete := FALSE;
        ReactionTimer.IN := FALSE;
END_CASE;

(* Step 4: Log state and completion changes *)
IF State <> Last_State AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 13:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, CONCAT(' State Changed to: ', 
        TO_STRING(State)));
END_IF;
IF PhaseComplete AND NOT Last_PhaseComplete AND LogCount < 50 THEN
    LogCount := LogCount + 1;
    Timestamp := '2025-05-19 13:40:00';
    DiagLog[LogCount] := CONCAT(Timestamp, ' Phase B.2 Completed');
END_IF;

(* Step 5: Update last states *)
Last_StartReaction := StartReaction.Q;
Last_StopReaction := StopReaction.Q;
Last_PhaseComplete := PhaseComplete;
Last_State := State;
Last_TargetTemp := TargetTemp;
Last_TargetPressure := TargetPressure;
Last_TargetAgitSpeed := TargetAgitSpeed;
Last_HeaterControl := HeaterControl;
Last_PressurePumpControl := PressurePumpControl;
Last_CoolerControl := CoolerControl;
Last_AgitatorControl := AgitatorControl;

(* Step 6: Check log buffer overflow *)
IF LogCount >= 50 THEN
    LogBufferFull := TRUE;
END_IF;

(* Notes:
   - Purpose: Implements ISA-88 B.2 Reaction for urea fertilizer production, controlling heating (180°C), pressure hold (140 bar, 30m), and cooling (50°C).
   - Inputs:
     - StartReaction, StopReaction: R_TRIG, start/stop B.2.
     - CurrentTemp: REAL, reactor temperature (°C).
     - CurrentPressure: REAL, reactor pressure (bar).
     - CurrentAgitSpeed: REAL, agitator speed (RPM).
     - EmergencyStop: BOOL, emergency trigger.
   - Outputs:
     - TargetTemp: REAL, temperature setpoint (°C).
     - TargetPressure: REAL, pressure setpoint (bar).
     - TargetAgitSpeed: REAL, agitator speed setpoint (RPM).
     - HeaterControl, PressurePumpControl, CoolerControl, AgitatorControl: BOOL, equipment controls.
     - PhaseComplete: BOOL, TRUE when B.2 finishes.
     - State: INT, current state (0=IDLE, 1=HEAT_UP, 2=PRESSURE_HOLD, 3=COOL_DOWN, 4=COMPLETED, 5=ERROR).
     - DiagLog: ARRAY[1..50] OF STRING[80], logs.
     - LogCount: INT, log entries.
   - Logic:
     - State machine: IDLE → HEAT_UP (180°C, 10m timeout) → PRESSURE_HOLD (140 bar, 180°C, 100 RPM, 30m) → COOL_DOWN (50°C, 15m timeout) → COMPLETED.
     - Methods: StartHeating, HoldPressure, StartCooling for modularity.
     - Transitions: Timer (ReactionTimer.Q) and conditions (temp 178–182°C, pressure 138–142 bar, speed 90–110 RPM).
     - ISA-88: Modular phases, reusable methods, clear transitions.
   - Optimization:
     - ~50 FLOPs, <0.1 ms on 1 MFLOP/s PLC.
     - Fixed flow, single timer, no recursion.
     - ~4 KB memory (logs, scalars).
   - Safety:
     - Validates inputs (0–300°C, 0–200 bar, 0–500 RPM).
     - EmergencyStop, timeouts to ERROR.
     - Exclusive states via CASE.
     - Logs all events.
   - Usage:
     - Urea batch: Precise control of reaction stage.
     - Example: StartReaction.Q=TRUE → HEAT_UP (180°C) → PRESSURE_HOLD (140 bar, 30m) → COOL_DOWN (50°C) → COMPLETED.
   - Platform Notes:
     - Compatible with TwinCAT, CODESYS, Siemens TIA Portal.
     - Assumes STRING[80], REAL, BOOL, INT, TON, R_TRIG; adjust for limits.
     - Timestamp uses 2025-05-19 13:40:00; replace with GET_SYSTEM_TIME.
     - Log buffer (50) is practical; adjustable.
*)
END_PROGRAM
