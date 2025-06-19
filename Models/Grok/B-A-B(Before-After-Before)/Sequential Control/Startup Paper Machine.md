(* IEC 61131-3 Structured Text: Valmet Paper Machine Press Section Startup *)
(* Purpose: Safely starts press section with synchronized rolls, nip pressure, and temperature *)

(* Function Block: Pre-Checks *)
FUNCTION_BLOCK PreCheck
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    GuardsClosed : BOOL;            (* TRUE if safety guards closed *)
    DrivesReady : BOOL;             (* TRUE if all drives operational *)
    NoAlarms : BOOL;               (* TRUE if no active alarms *)
END_VAR
VAR_OUTPUT
    Complete : BOOL;                (* TRUE when checks pass *)
    SafetyFault : BOOL;             (* TRUE if safety condition fails *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Checking, 2=Complete *)
    CheckTimer : TON;               (* Timer for pre-check duration *)
END_VAR
IF NOT Enable THEN
    State := 0;
    Complete := FALSE;
    SafetyFault := FALSE;
    CheckTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            Complete := FALSE;
            SafetyFault := FALSE;
            State := 1;
            CheckTimer(IN := TRUE, PT := T#30s);
        1: (* Checking *)
            IF NOT GuardsClosed OR NOT DrivesReady OR NOT NoAlarms THEN
                SafetyFault := TRUE;
                State := 0;
            ELSIF CheckTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            Complete := TRUE;
            SafetyFault := FALSE;
            CheckTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Verifies safety conditions before startup *)
END_FUNCTION_BLOCK

(* Function Block: Activate Rolls *)
FUNCTION_BLOCK ActivateRolls
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    RollSpeed1_PV : REAL;           (* Measured speed of roll 1 in m/min *)
    RollSpeed2_PV : REAL;           (* Measured speed of roll 2 in m/min *)
    ConveyorSpeed_PV : REAL;        (* Measured conveyor speed in m/min *)
END_VAR
VAR_OUTPUT
    RollDriveOn : BOOL;             (* TRUE to activate roll drives *)
    ConveyorDriveOn : BOOL;         (* TRUE to activate conveyor drive *)
    SpeedSetpoint : REAL;           (* Initial speed setpoint in m/min *)
    SyncFault : BOOL;               (* TRUE if speeds not synchronized *)
    Complete : BOOL;                (* TRUE when rolls activated *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Starting, 2=Complete *)
    StartTimer : TON;               (* Timer for roll activation *)
    InitialSpeed : REAL := 100.0;   (* Initial speed: 100 m/min *)
    SyncTol : REAL := 5.0;          (* Speed sync tolerance: ±5 m/min *)
END_VAR
IF NOT Enable THEN
    State := 0;
    RollDriveOn := FALSE;
    ConveyorDriveOn := FALSE;
    SpeedSetpoint := 0.0;
    SyncFault := FALSE;
    Complete := FALSE;
    StartTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            RollDriveOn := FALSE;
            ConveyorDriveOn := FALSE;
            SpeedSetpoint := 0.0;
            SyncFault := FALSE;
            Complete := FALSE;
            State := 1;
            StartTimer(IN := TRUE, PT := T#1m);
        1: (* Starting *)
            RollDriveOn := TRUE;
            ConveyorDriveOn := TRUE;
            SpeedSetpoint := InitialSpeed;
            SyncFault := (ABS(RollSpeed1_PV - RollSpeed2_PV) > SyncTol) OR 
                         (ABS(RollSpeed1_PV - ConveyorSpeed_PV) > SyncTol);
            IF StartTimer.Q AND NOT SyncFault THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            RollDriveOn := TRUE;
            ConveyorDriveOn := TRUE;
            SpeedSetpoint := InitialSpeed;
            Complete := TRUE;
            StartTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Starts rolls and conveyors at 100 m/min, ensures synchronization *)
END_FUNCTION_BLOCK

(* Function Block: Build Pressure *)
FUNCTION_BLOCK BuildPressure
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    NipPressure_PV : REAL;          (* Measured nip pressure in kN/m *)
END_VAR
VAR_OUTPUT
    PressureSetpoint : REAL;        (* Nip pressure setpoint in kN/m *)
    PressureFault : BOOL;           (* TRUE if pressure deviates *)
    Complete : BOOL;                (* TRUE when pressure reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Ramping, 2=Complete *)
    RampTimer : TON;                (* Timer for pressure ramp *)
    TargetPressure : REAL := 250.0; (* Target pressure: 250 kN/m *)
    RampDuration : TIME := T#5m;    (* Ramp duration: 5 min *)
    PressureTol : REAL := 10.0;     (* Pressure tolerance: ±10 kN/m *)
    PressureRate : REAL;            (* Ramp rate in kN/m per second *)
    ElapsedTime : TIME;             (* Elapsed time since ramp start *)
END_VAR
IF NOT Enable THEN
    State := 0;
    PressureSetpoint := 0.0;
    PressureFault := FALSE;
    Complete := FALSE;
    RampTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            PressureSetpoint := 0.0;
            PressureFault := FALSE;
            Complete := FALSE;
            PressureRate := TargetPressure / TIME_TO_REAL(RampDuration, 1.0);
            State := 1;
            RampTimer(IN := TRUE, PT := RampDuration);
        1: (* Ramping *)
            ElapsedTime := RampTimer.ET;
            PressureSetpoint := PressureRate * TIME_TO_REAL(ElapsedTime, 1.0);
            IF PressureSetpoint > TargetPressure THEN
                PressureSetpoint := TargetPressure;
            END_IF;
            PressureFault := ABS(NipPressure_PV - PressureSetpoint) > PressureTol;
            IF RampTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            PressureSetpoint := TargetPressure;
            PressureFault := ABS(NipPressure_PV - TargetPressure) > PressureTol;
            Complete := TRUE;
            RampTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Gradually builds nip pressure to 250 kN/m *)
END_FUNCTION_BLOCK

(* Function Block: Control Temperature *)
FUNCTION_BLOCK ControlTemperature
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    Temp_PV : REAL;                 (* Measured felt temperature in °C *)
END_VAR
VAR_OUTPUT
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    TempFault : BOOL;               (* TRUE if temperature deviates *)
    Complete : BOOL;                (* TRUE when temperature reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Heating, 2=Complete *)
    TempTimer : TON;                (* Timer for temperature stabilization *)
    TargetTemp : REAL := 90.0;      (* Target temperature: 90°C *)
    TempTol : REAL := 3.0;          (* Temperature tolerance: ±3°C *)
END_VAR
IF NOT Enable THEN
    State := 0;
    HeaterOn := FALSE;
    TempFault := FALSE;
    Complete := FALSE;
    TempTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            HeaterOn := FALSE;
            TempFault := FALSE;
            Complete := FALSE;
            State := 1;
            TempTimer(IN := TRUE, PT := T#5m);
        1: (* Heating *)
            HeaterOn := TRUE;
            TempFault := ABS(Temp_PV - TargetTemp) > TempTol;
            IF ABS(Temp_PV - TargetTemp) <= TempTol AND TempTimer.Q THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            HeaterOn := ABS(Temp_PV - TargetTemp) > TempTol;
            TempFault := ABS(Temp_PV - TargetTemp) > TempTol;
            Complete := TRUE;
            TempTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Heats press felts to 90°C for optimal pressing *)
END_FUNCTION_BLOCK

(* Function Block: Ramp Speed *)
FUNCTION_BLOCK RampSpeed
VAR_INPUT
    Enable : BOOL;                  (* TRUE to run operation *)
    RollSpeed1_PV : REAL;           (* Measured speed of roll 1 in m/min *)
    RollSpeed2_PV : REAL;           (* Measured speed of roll 2 in m/min *)
    ConveyorSpeed_PV : REAL;        (* Measured conveyor speed in m/min *)
END_VAR
VAR_OUTPUT
    SpeedSetpoint : REAL;           (* Speed setpoint in m/min *)
    SyncFault : BOOL;               (* TRUE if speeds not synchronized *)
    Complete : BOOL;                (* TRUE when speed reached *)
END_VAR
VAR
    State : INT := 0;               (* 0=Idle, 1=Ramping, 2=Complete *)
    RampTimer : TON;                (* Timer for speed ramp *)
    InitialSpeed : REAL := 100.0;   (* Initial speed: 100 m/min *)
    TargetSpeed : REAL := 600.0;    (* Target speed: 600 m/min *)
    RampDuration : TIME := T#10m;   (* Ramp duration: 10 min *)
    SpeedRate : REAL;               (* Ramp rate in m/min per second *)
    ElapsedTime : TIME;             (* Elapsed time since ramp start *)
    SyncTol : REAL := 5.0;          (* Speed sync tolerance: ±5 m/min *)
END_VAR
IF NOT Enable THEN
    State := 0;
    SpeedSetpoint := InitialSpeed;
    SyncFault := FALSE;
    Complete := FALSE;
    RampTimer(IN := FALSE);
ELSE
    CASE State OF
        0: (* Idle *)
            SpeedSetpoint := InitialSpeed;
            SyncFault := FALSE;
            Complete := FALSE;
            SpeedRate := (TargetSpeed - InitialSpeed) / TIME_TO_REAL(RampDuration, 1.0);
            State := 1;
            RampTimer(IN := TRUE, PT := RampDuration);
        1: (* Ramping *)
            ElapsedTime := RampTimer.ET;
            SpeedSetpoint := InitialSpeed + (SpeedRate * TIME_TO_REAL(ElapsedTime, 1.0));
            IF SpeedSetpoint > TargetSpeed THEN
                SpeedSetpoint := TargetSpeed;
            END_IF;
            SyncFault := (ABS(RollSpeed1_PV - RollSpeed2_PV) > SyncTol) OR 
                         (ABS(RollSpeed1_PV - ConveyorSpeed_PV) > SyncTol);
            IF RampTimer.Q AND NOT SyncFault THEN
                State := 2;
            END_IF;
        2: (* Complete *)
            SpeedSetpoint := TargetSpeed;
            SyncFault := (ABS(RollSpeed1_PV - RollSpeed2_PV) > SyncTol) OR 
                         (ABS(RollSpeed1_PV - ConveyorSpeed_PV) > SyncTol);
            Complete := TRUE;
            RampTimer(IN := FALSE);
    END_CASE;
END_IF;
(* Ramps roll speed to 600 m/min, ensures synchronization *)
END_FUNCTION_BLOCK

(* Main Program: Paper Machine Press Section Startup *)
PROGRAM PaperMachinePressStartup
VAR
    (* Inputs *)
    StartStartup : BOOL;             (* TRUE to start sequence *)
    EmergencyStop : BOOL;           (* TRUE to halt operations *)
    GuardsClosed : BOOL;            (* TRUE if safety guards closed *)
    DrivesReady : BOOL;             (* TRUE if all drives operational *)
    NoAlarms : BOOL;                (* TRUE if no active alarms *)
    RollSpeed1_PV : REAL;           (* Measured speed of roll 1 in m/min *)
    RollSpeed2_PV : REAL;           (* Measured speed of roll 2 in m/min *)
    ConveyorSpeed_PV : REAL;        (* Measured conveyor speed in m/min *)
    NipPressure_PV : REAL;          (* Measured nip pressure in kN/m *)
    FeltTemp_PV : REAL;             (* Measured felt temperature in °C *)
    
    (* Outputs *)
    RollDriveOn : BOOL;             (* TRUE to activate roll drives *)
    ConveyorDriveOn : BOOL;         (* TRUE to activate conveyor drive *)
    SpeedSetpoint : REAL;           (* Speed setpoint in m/min *)
    PressureSetpoint : REAL;        (* Nip pressure setpoint in kN/m *)
    HeaterOn : BOOL;                (* TRUE to activate heater *)
    StartupComplete : BOOL;         (* TRUE when startup complete *)
    SafetyFault : BOOL;             (* TRUE if safety condition fails *)
    SyncFault : BOOL;               (* TRUE if speeds not synchronized *)
    PressureFault : BOOL;           (* TRUE if pressure deviates *)
    TempFault : BOOL;               (* TRUE if temperature deviates *)
    
    (* State Machine *)
    State : INT := 0;               (* Main state *)
    CONSTANT
        STATE_IDLE : INT := 0;      (* Waiting to start *)
        STATE_PRECHECK : INT := 1;  (* Safety and system checks *)
        STATE_ROLL_ACTIVATION : INT := 2; (* Activate rolls/conveyors *)
        STATE_PRESSURE_BUILDUP : INT := 3; (* Build nip pressure *)
        STATE_TEMPERATURE_CONTROL : INT := 4; (* Control felt temperature *)
        STATE_SPEED_RAMPUP : INT := 5; (* Ramp up speed *)
        STATE_FULL_OPERATION : INT := 6; (* Full production *)
    END_CONSTANT
    
    (* Operation Instances *)
    PreCheckOp : PreCheck;          (* Pre-checks *)
    RollOp : ActivateRolls;         (* Roll activation *)
    PressureOp : BuildPressure;     (* Pressure build-up *)
    TempOp : ControlTemperature;    (* Temperature control *)
    SpeedOp : RampSpeed;            (* Speed ramp-up *)
    
    (* Diagnostics *)
    CurrentPhase : STRING[80];      (* Current phase for HMI *)
END_VAR

(* Main Logic *)
IF EmergencyStop THEN
    State := STATE_IDLE;
    RollDriveOn := FALSE;
    ConveyorDriveOn := FALSE;
    SpeedSetpoint := 0.0;
    PressureSetpoint := 0.0;
    HeaterOn := FALSE;
    StartupComplete := FALSE;
    SafetyFault := FALSE;
    SyncFault := FALSE;
    PressureFault := FALSE;
    TempFault := FALSE;
    PreCheckOp.Enable := FALSE;
    RollOp.Enable := FALSE;
    PressureOp.Enable := FALSE;
    TempOp.Enable := FALSE;
    SpeedOp.Enable := FALSE;
    CurrentPhase := 'Emergency Stop';
ELSIF StartStartup AND State = STATE_IDLE THEN
    State := STATE_PRECHECK;
    StartupComplete := FALSE;
    CurrentPhase := 'Pre-Checks';
END_IF;

(* State Machine *)
CASE State OF
    STATE_IDLE:
        RollDriveOn := FALSE;
        ConveyorDriveOn := FALSE;
        SpeedSetpoint := 0.0;
        PressureSetpoint := 0.0;
        HeaterOn := FALSE;
        StartupComplete := FALSE;
        SafetyFault := FALSE;
        SyncFault := FALSE;
        PressureFault := FALSE;
        TempFault := FALSE;
        PreCheckOp.Enable := FALSE;
        RollOp.Enable := FALSE;
        PressureOp.Enable := FALSE;
        TempOp.Enable := FALSE;
        SpeedOp.Enable := FALSE;
        CurrentPhase := 'Idle';
        (* Idle: All systems off, ready for startup *)

    STATE_PRECHECK:
        PreCheckOp(Enable := TRUE, GuardsClosed := GuardsClosed, DrivesReady := DrivesReady, NoAlarms := NoAlarms);
        SafetyFault := PreCheckOp.SafetyFault;
        CurrentPhase := 'Pre-Checks';
        IF PreCheckOp.Complete THEN
            State := STATE_ROLL_ACTIVATION;
            PreCheckOp.Enable := FALSE;
        ELSIF SafetyFault THEN
            State := STATE_IDLE;
            PreCheckOp.Enable := FALSE;
        END_IF;
        (* Verifies safety and system readiness *)

    STATE_ROLL_ACTIVATION:
        RollOp(Enable := TRUE, RollSpeed1_PV := RollSpeed1_PV, RollSpeed2_PV := RollSpeed2_PV, ConveyorSpeed_PV := ConveyorSpeed_PV);
        RollDriveOn := RollOp.RollDriveOn;
        ConveyorDriveOn := RollOp.ConveyorDriveOn;
        SpeedSetpoint := RollOp.SpeedSetpoint;
        SyncFault := RollOp.SyncFault;
        CurrentPhase := 'Roll Activation';
        IF RollOp.Complete THEN
            State := STATE_PRESSURE_BUILDUP;
            RollOp.Enable := FALSE;
        ELSIF SyncFault THEN
            State := STATE_IDLE;
            RollOp.Enable := FALSE;
        END_IF;
        (* Starts rolls/conveyors at 100 m/min *)

    STATE_PRESSURE_BUILDUP:
        PressureOp(Enable := TRUE, NipPressure_PV := NipPressure_PV);
        PressureSetpoint := PressureOp.PressureSetpoint;
        PressureFault := PressureOp.PressureFault;
        CurrentPhase := 'Pressure Build-Up';
        IF PressureOp.Complete THEN
            State := STATE_TEMPERATURE_CONTROL;
            PressureOp.Enable := FALSE;
        ELSIF PressureFault THEN
            State := STATE_IDLE;
            PressureOp.Enable := FALSE;
        END_IF;
        (* Builds nip pressure to 250 kN/m *)

    STATE_TEMPERATURE_CONTROL:
        TempOp(Enable := TRUE, Temp_PV := FeltTemp_PV);
        HeaterOn := TempOp.HeaterOn;
        TempFault := TempOp.TempFault;
        CurrentPhase := 'Temperature Control';
        IF TempOp.Complete THEN
            State := STATE_SPEED_RAMPUP;
            TempOp.Enable := FALSE;
        ELSIF TempFault THEN
            State := STATE_IDLE;
            TempOp.Enable := FALSE;
        END_IF;
        (* Heats felts to 90°C *)

    STATE_SPEED_RAMPUP:
        SpeedOp(Enable := TRUE, RollSpeed1_PV := RollSpeed1_PV, RollSpeed2_PV := RollSpeed2_PV, ConveyorSpeed_PV := ConveyorSpeed_PV);
        SpeedSetpoint := SpeedOp.SpeedSetpoint;
        SyncFault := SpeedOp.SyncFault;
        CurrentPhase := 'Speed Ramp-Up';
        IF SpeedOp.Complete THEN
            State := STATE_FULL_OPERATION;
            SpeedOp.Enable := FALSE;
        ELSIF SyncFault THEN
            State := STATE_IDLE;
            SpeedOp.Enable := FALSE;
        END_IF;
        (* Ramps speed to 600 m/min *)

    STATE_FULL_OPERATION:
        RollDriveOn := TRUE;
        ConveyorDriveOn := TRUE;
        SpeedSetpoint := 600.0;
        PressureSetpoint := 250.0;
        HeaterOn := ABS(FeltTemp_PV - 90.0) > 3.0;
        StartupComplete := TRUE;
        CurrentPhase := 'Full Operation';
        SyncFault := (ABS(RollSpeed1_PV - RollSpeed2_PV) > 5.0) OR 
                     (ABS(RollSpeed1_PV - ConveyorSpeed_PV) > 5.0);
        PressureFault := ABS(NipPressure_PV - 250.0) > 10.0;
        TempFault := ABS(FeltTemp_PV - 90.0) > 3.0;
        IF SyncFault OR PressureFault OR TempFault THEN
            State := STATE_IDLE;
        END_IF;
        (* Maintains full production with ongoing checks *)

ELSE
    State := STATE_IDLE; (* Fallback to safe state *)
END_CASE;

(* Notes:
   - ISA-88 Compliance:
     - Procedure: Press section startup with Unit Procedures (e.g., Roll Activation).
     - Operations: PreCheck, ActivateRolls, BuildPressure, ControlTemperature, RampSpeed.
     - Recipe Parameters: SpeedSetpoint, PressureSetpoint, TargetTemp for flexibility.
     - State Model: Idle, Running, Complete per ISA-88.
   - Startup Sequence:
     1. Pre-Checks: Verify guards, drives, alarms (30 s).
     2. Roll Activation: Start rolls/conveyors at 100 m/min (1 min).
     3. Pressure Build-Up: Ramp to 250 kN/m (5 min).
     4. Temperature Control: Heat to 90°C (5 min).
     5. Speed Ramp-Up: Increase to 600 m/min (10 min).
     6. Full Operation: Stabilize production.
   - Safety:
     - EmergencyStop halts all operations.
     - Interlocks: SafetyFault (guards/drives), SyncFault (±5 m/min), PressureFault (±10 kN/m), TempFault (±3°C).
   - Quality:
     - Synchronized rolls prevent sheet tearing.
     - Proper nip pressure (250 kN/m) ensures uniform sheet thickness, prevents felt deformation.
     - Stable 90°C prevents sheet wrinkling or sticking.
   - Physical Integration:
     - Inputs: StartStartup, GuardsClosed, DrivesReady, NoAlarms, Speed_PV, Pressure_PV, Temp_PV.
     - Outputs: RollDriveOn, ConveyorDriveOn, SpeedSetpoint, PressureSetpoint, HeaterOn.
   - Scalability:
     - Modular blocks support different machine sizes.
     - Parameters adjustable for varying paper grades.
   - Maintenance:
     - Function blocks simplify debugging.
     - HMI displays CurrentPhase, Faults for diagnostics.
*)
END_PROGRAM
