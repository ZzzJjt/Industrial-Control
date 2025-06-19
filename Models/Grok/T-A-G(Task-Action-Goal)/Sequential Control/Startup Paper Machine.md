(* IEC 61131-3 Structured Text Program for Valmet Paper Machine Press Section Startup *)
(* Implements safe, sequenced startup with speed, pressure, and temperature control *)
(* Conforms to industrial automation best practices *)

PROGRAM PressSectionStartupControl
VAR
    (* Phase State *)
    PhaseState: INT := 0; (* 0: Idle, 1: PreStart, 2: ConveyorStart, 3: RollStart, 4: TempRegulation, 5: SpeedRamp, 6: PressureRamp, 7: SyncStabilize, 8: Running, 9: Fault *)
    FaultState: BOOL := FALSE;
    EmergencyStop: BOOL := FALSE;
    
    (* Process Parameters *)
    TargetRollSpeed: REAL := 500.0; (* Target roll speed in m/min *)
    TargetNipPressure: REAL := 250.0; (* Target nip pressure in kN/m *)
    TempMinSetpoint: REAL := 85.0; (* Minimum roll temperature in °C *)
    TempMaxSetpoint: REAL := 90.0; (* Maximum roll temperature in °C *)
    TempTolerance: REAL := 1.0; (* Acceptable temperature deviation *)
    SpeedRampTime: TIME := T#10m; (* Speed ramp-up duration *)
    PressureRampTime: TIME := T#5m; (* Pressure ramp-up duration *)
    SyncTolerance: REAL := 0.01; (* Max speed difference as fraction, 1% *)
    MaxStartupTime: TIME := T#20m; (* Maximum startup duration *)
    
    (* Process Inputs *)
    CurrentRollSpeed: REAL; (* Current press roll speed in m/min *)
    CurrentConveyorSpeed: REAL; (* Current conveyor speed in m/min *)
    CurrentNipPressure: REAL; (* Current nip pressure in kN/m *)
    CurrentTemp: REAL; (* Current roll temperature in °C *)
    WebThreaded: BOOL; (* Paper web properly threaded *)
    SafetyClear: BOOL; (* Safety systems clear *)
    
    (* Process Outputs *)
    ConveyorMotorOn: BOOL; (* Conveyor motor control *)
    RollMotorOn: BOOL; (* Press roll motor control *)
    RollSpeedSetpoint: REAL; (* Roll speed setpoint in m/min *)
    ConveyorSpeedSetpoint: REAL; (* Conveyor speed setpoint in m/min *)
    NipPressureSetpoint: REAL; (* Nip pressure setpoint in kN/m *)
    HeaterOn: BOOL; (* Roll heating system control *)
    StartupComplete: BOOL; (* Startup completion flag *)
    
    (* Timers *)
    PhaseTimer: TON; (* Timer for overall startup duration *)
    SpeedRampTimer: TON; (* Timer for speed ramp-up *)
    PressureRampTimer: TON; (* Timer for pressure ramp-up *)
    TransitionDelay: TON; (* Delay for phase transitions *)
    
    (* Configuration *)
    TransitionDelayTime: TIME := T#5s; (* Delay between phase transitions *)
    RampUpdateInterval: TIME := T#10s; (* Speed/pressure update interval *)
    InitialSpeed: REAL := 50.0; (* Initial speed for conveyors and rolls *)
END_VAR

(* Modular Function Blocks *)
FUNCTION_BLOCK ActivateConveyor
    VAR_INPUT
        SafetyClear: BOOL; (* Safety systems status *)
        WebThreaded: BOOL; (* Web threading status *)
    END_VAR
    VAR_OUTPUT
        MotorCmd: BOOL; (* Conveyor motor command *)
        ConveyorReady: BOOL; (* Conveyor activation complete *)
    END_VAR
    (* Logic to start conveyors at initial speed *)
    MotorCmd := SafetyClear AND WebThreaded;
    ConveyorReady := MotorCmd;
END_FUNCTION_BLOCK

FUNCTION_BLOCK ActivateRolls
    VAR_INPUT
        ConveyorRunning: BOOL; (* Conveyor status *)
        SafetyClear: BOOL; (* Safety systems status *)
    END_VAR
    VAR_OUTPUT
        MotorCmd: BOOL; (* Roll motor command *)
        RollsReady: BOOL; (* Rolls activation complete *)
    END_VAR
    (* Logic to start press rolls at initial speed *)
    MotorCmd := ConveyorRunning AND SafetyClear;
    RollsReady := MotorCmd;
END_FUNCTION_BLOCK

FUNCTION_BLOCK RegulateTemperature
    VAR_INPUT
        CurrentTemp: REAL; (* Measured temperature *)
        TempMin: REAL; (* Minimum target temperature *)
        TempMax: REAL; (* Maximum target temperature *)
        Tolerance: REAL; (* Acceptable deviation *)
    END_VAR
    VAR_OUTPUT
        HeaterCmd: BOOL; (* Heater control *)
        TempStable: BOOL; (* Temperature within range *)
    END_VAR
    (* Logic to maintain roll temperature at 85–90°C *)
    HeaterCmd := (CurrentTemp < (TempMin - Tolerance));
    TempStable := (CurrentTemp >= TempMin) AND (CurrentTemp <= TempMax);
END_FUNCTION_BLOCK

FUNCTION_BLOCK RampSpeed
    VAR_INPUT
        InitialSpeed: REAL; (* Starting speed *)
        TargetSpeed: REAL; (* Final speed *)
        CurrentSpeed: REAL; (* Measured speed *)
        RampDuration: TIME; (* Ramp duration *)
        Timer: TON; (* Ramp timer *)
        Interval: TIME; (* Update interval *)
    END_VAR
    VAR_OUTPUT
        SpeedSetpoint: REAL; (* Calculated speed setpoint *)
        RampComplete: BOOL; (* Ramp completion flag *)
        Fault: BOOL; (* Fault flag *)
    END_VAR
    VAR
        ElapsedTime: TIME; (* Time elapsed *)
        SpeedIncrement: REAL; (* Total speed increase *)
        SpeedPerInterval: REAL; (* Speed increase per interval *)
    END_VAR
    (* Logic to linearly ramp up speed *)
    Timer(IN := TRUE, PT := RampDuration);
    ElapsedTime := Timer.ET;
    SpeedIncrement := TargetSpeed - InitialSpeed;
    SpeedPerInterval := SpeedIncrement / (TIME_TO_REAL(RampDuration) / TIME_TO_REAL(Interval));
    
    SpeedSetpoint := InitialSpeed + (SpeedPerInterval * (TIME_TO_REAL(ElapsedTime) / TIME_TO_REAL(Interval)));
    SpeedSetpoint := MIN(TargetSpeed, MAX(InitialSpeed, SpeedSetpoint));
    
    RampComplete := Timer.Q OR (ABS(SpeedSetpoint - TargetSpeed) < 0.1);
    Fault := (ABS(CurrentSpeed - SpeedSetpoint) > (SpeedSetpoint * 0.05)) AND NOT RampComplete;
END_FUNCTION_BLOCK

FUNCTION_BLOCK RampPressure
    VAR_INPUT
        InitialPressure: REAL; (* Starting pressure *)
        TargetPressure: REAL; (* Final pressure *)
        CurrentPressure: REAL; (* Measured pressure *)
        RampDuration: TIME; (* Ramp duration *)
        Timer: TON; (* Ramp timer *)
        Interval: TIME; (* Update interval *)
    END_VAR
    VAR_OUTPUT
        PressureSetpoint: REAL; (* Calculated pressure setpoint *)
        RampComplete: BOOL; (* Ramp completion flag *)
        Fault: BOOL; (* Fault flag *)
    END_VAR
    VAR
        ElapsedTime: TIME; (* Time elapsed *)
        PressureIncrement: REAL; (* Total pressure increase *)
        PressurePerInterval: REAL; (* Pressure increase per interval *)
    END_VAR
    (* Logic to linearly ramp up nip pressure *)
    Timer(IN := TRUE, PT := RampDuration);
    ElapsedTime := Timer.ET;
    PressureIncrement := TargetPressure - InitialPressure;
    PressurePerInterval := PressureIncrement / (TIME_TO_REAL(RampDuration) / TIME_TO_REAL(Interval));
    
    PressureSetpoint := InitialPressure + (PressurePerInterval * (TIME_TO_REAL(ElapsedTime) / TIME_TO_REAL(Interval)));
    PressureSetpoint := MIN(TargetPressure, MAX(InitialPressure, PressureSetpoint));
    
    RampComplete := Timer.Q OR (ABS(PressureSetpoint - TargetPressure) < 0.1);
    Fault := (ABS(CurrentPressure - PressureSetpoint) > (PressureSetpoint * 0.05)) AND NOT RampComplete;
END_FUNCTION_BLOCK

FUNCTION_BLOCK SynchronizeSpeeds
    VAR_INPUT
        RollSpeed: REAL; (* Current roll speed *)
        ConveyorSpeed: REAL; (* Current conveyor speed *)
        Tolerance: REAL; (* Max allowable speed difference fraction *)
    END_VAR
    VAR_OUTPUT
        SyncComplete: BOOL; (* Synchronization complete *)
        Fault: BOOL; (* Synchronization fault *)
    END_VAR
    (* Logic to ensure roll and conveyor speeds are synchronized *)
    SyncComplete := (ABS(RollSpeed - ConveyorSpeed) / RollSpeed) <= Tolerance;
    Fault := NOT SyncComplete;
END_FUNCTION_BLOCK

(* Main Control Logic *)
VAR
    ConveyorFB: ActivateConveyor;
    RollsFB: ActivateRolls;
    TempFB: RegulateTemperature;
    SpeedRampFB: RampSpeed;
    PressureRampFB: RampPressure;
    SyncFB: SynchronizeSpeeds;
END_VAR

CASE PhaseState OF
    0: (* Idle State *)
        (* Wait for start command, ensure no faults or emergency stop *)
        IF StartCommand AND NOT EmergencyStop AND NOT FaultState THEN
            PhaseState := 1;
            PhaseTimer(IN := TRUE, PT := MaxStartupTime);
        END_IF;
    
    1: (* Pre-Start Checks *)
        (* Verify safety and web threading *)
        SafetyClear := NOT EmergencyStop AND NOT FaultState;
        
        (* Transition to conveyor start *)
        IF SafetyClear AND WebThreaded AND TransitionDelay.Q THEN
            PhaseState := 2;
            TransitionDelay(IN := FALSE);
        ELSIF SafetyClear AND WebThreaded THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    2: (* Conveyor Start *)
        (* Activate conveyors at initial speed *)
        ConveyorFB(SafetyClear := SafetyClear, WebThreaded := WebThreaded);
        ConveyorMotorOn := ConveyorFB.MotorCmd;
        ConveyorSpeedSetpoint := InitialSpeed;
        
        (* Transition to roll start *)
        IF ConveyorFB.ConveyorReady AND TransitionDelay.Q THEN
            PhaseState := 3;
            TransitionDelay(IN := FALSE);
        ELSIF ConveyorFB.ConveyorReady THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    3: (* Roll Start *)
        (* Activate press rolls at initial speed *)
        RollsFB(ConveyorRunning := ConveyorFB.ConveyorReady, SafetyClear := SafetyClear);
        RollMotorOn := RollsFB.MotorCmd;
        RollSpeedSetpoint := InitialSpeed;
        
        (* Transition to temperature regulation *)
        IF RollsFB.RollsReady AND TransitionDelay.Q THEN
            PhaseState := 4;
            TransitionDelay(IN := FALSE);
        ELSIF RollsFB.RollsReady THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    4: (* Temperature Regulation *)
        (* Regulate roll temperature to 85–90°C *)
        TempFB(CurrentTemp := CurrentTemp, 
               TempMin := TempMinSetpoint, 
               TempMax := TempMaxSetpoint, 
               Tolerance := TempTolerance);
        HeaterOn := TempFB.HeaterCmd;
        
        (* Transition to speed ramp-up *)
        IF TempFB.TempStable AND TransitionDelay.Q THEN
            PhaseState := 5;
            TransitionDelay(IN := FALSE);
        ELSIF TempFB.TempStable THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    5: (* Speed Ramp-Up *)
        (* Ramp roll and conveyor speed to 500 m/min *)
        SpeedRampFB(InitialSpeed := InitialSpeed, 
                    TargetSpeed := TargetRollSpeed, 
                    CurrentSpeed := CurrentRollSpeed, 
                    RampDuration := SpeedRampTime, 
                    Timer := SpeedRampTimer, 
                    Interval := RampUpdateInterval);
        RollSpeedSetpoint := SpeedRampFB.SpeedSetpoint;
        ConveyorSpeedSetpoint := SpeedRampFB.SpeedSetpoint;
        
        (* Fault check *)
        IF SpeedRampFB.Fault THEN
            FaultState := TRUE;
            PhaseState := 9;
        END_IF;
        
        (* Transition to pressure ramp-up *)
        IF SpeedRampFB.RampComplete AND TransitionDelay.Q THEN
            PhaseState := 6;
            TransitionDelay(IN := FALSE);
        ELSIF SpeedRampFB.RampComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    6: (* Pressure Ramp-Up *)
        (* Ramp nip pressure to 250 kN/m *)
        PressureRampFB(InitialPressure := 0.0, 
                       TargetPressure := TargetNipPressure, 
                       CurrentPressure := CurrentNipPressure, 
                       RampDuration := PressureRampTime, 
                       Timer := PressureRampTimer, 
                       Interval := RampUpdateInterval);
        NipPressureSetpoint := PressureRampFB.PressureSetpoint;
        
        (* Fault check *)
        IF PressureRampFB.Fault THEN
            FaultState := TRUE;
            PhaseState := 9;
        END_IF;
        
        (* Transition to synchronization *)
        IF PressureRampFB.RampComplete AND TransitionDelay.Q THEN
            PhaseState := 7;
            TransitionDelay(IN := FALSE);
        ELSIF PressureRampFB.RampComplete THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    7: (* Synchronization and Stabilization *)
        (* Ensure roll and conveyor speeds are synchronized *)
        SyncFB(RollSpeed := CurrentRollSpeed, 
               ConveyorSpeed := CurrentConveyorSpeed, 
               Tolerance := SyncTolerance);
        
        (* Maintain temperature *)
        TempFB(CurrentTemp := CurrentTemp, 
               TempMin := TempMinSetpoint, 
               TempMax := TempMaxSetpoint, 
               Tolerance := TempTolerance);
        HeaterOn := TempFB.HeaterCmd;
        
        (* Fault check *)
        IF SyncFB.Fault OR NOT TempFB.TempStable THEN
            FaultState := TRUE;
            PhaseState := 9;
        END_IF;
        
        (* Transition to running *)
        IF SyncFB.SyncComplete AND TempFB.TempStable AND TransitionDelay.Q THEN
            PhaseState := 8;
            TransitionDelay(IN := FALSE);
            StartupComplete := TRUE;
        ELSIF SyncFB.SyncComplete AND TempFB.TempStable THEN
            TransitionDelay(IN := TRUE, PT := TransitionDelayTime);
        END_IF;
    
    8: (* Running State *)
        (* Maintain normal operation *)
        RollSpeedSetpoint := TargetRollSpeed;
        ConveyorSpeedSetpoint := TargetRollSpeed;
        NipPressureSetpoint := TargetNipPressure;
        TempFB(CurrentTemp := CurrentTemp, 
               TempMin := TempMinSetpoint, 
               TempMax := TempMaxSetpoint, 
               Tolerance := TempTolerance);
        HeaterOn := TempFB.HeaterCmd;
        
        (* Monitor for faults or stop command *)
        IF StopCommand OR EmergencyStop OR FaultState THEN
            PhaseState := 9;
        END_IF;
    
    9: (* Fault State *)
        (* Disable all outputs *)
        ConveyorMotorOn := FALSE;
        RollMotorOn := FALSE;
        RollSpeedSetpoint := 0.0;
        ConveyorSpeedSetpoint := 0.0;
        NipPressureSetpoint := 0.0;
        HeaterOn := FALSE;
        PhaseTimer(IN := FALSE);
        SpeedRampTimer(IN := FALSE);
        PressureRampTimer(IN := FALSE);
        
        (* Require manual reset *)
        IF ResetCommand AND NOT EmergencyStop THEN
            PhaseState := 0;
            FaultState := FALSE;
            StartupComplete := FALSE;
        END_IF;
END_CASE;

(* Safety and Fault Monitoring *)
IF EmergencyStop THEN
    PhaseState := 9;
    FaultState := TRUE;
END_IF;

(* Parameter Monitoring *)
IF (CurrentTemp > (TempMaxSetpoint + 5.0) OR 
    CurrentTemp < (TempMinSetpoint - 5.0)) AND PhaseState IN (4..8) THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

IF (CurrentNipPressure > (TargetNipPressure + 10.0)) AND PhaseState IN (6..8) THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

IF (ABS(CurrentRollSpeed - CurrentConveyorSpeed) / CurrentRollSpeed > SyncTolerance) AND PhaseState IN (5..8) THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

(* Phase Timeout Check *)
IF PhaseTimer.Q THEN
    FaultState := TRUE;
    PhaseState := 9;
END_IF;

END_PROGRAM
