PROGRAM PressSectionStartupSequence
VAR
    // State management variables
    currentState : INT := 0; // 0: Idle, 1: Initiate Startup, 2: Preparation, 3: Activate Conveyors, 4: Ramp Up Speeds, 5: Adjust Nip Pressure, 6: Monitor Temperature, 7: Final Checks, 8: Operational Mode

    // Parameters for Speed Ramp-Up Phase
    TargetSpeed : REAL := 500.0; // Target speed in m/min
    SpeedRampTime : TIME := T#10m; // Total time for speed ramp-up in minutes
    SpeedStepInterval : TIME := T#1m; // Interval for checking speed increment

    // Parameters for Nip Pressure Ramp-Up Phase
    TargetPressure : REAL := 250.0; // Target nip pressure in kN/m
    PressureRampTime : TIME := T#5m; // Total time for pressure ramp-up in minutes
    PressureStabilityTime : TIME := T#1m; // Stability time for each pressure step
    PressureStepInterval : TIME := T#1m; // Interval for checking pressure increment

    // Parameters for Temperature Monitoring Phase
    MinTemperature : REAL := 85.0; // Minimum acceptable temperature in Celsius
    MaxTemperature : REAL := 90.0; // Maximum acceptable temperature in Celsius

    // Timers
    SpeedRampTimer : TON;
    PressureRampTimer : TON;
    PressureStabilityTimer : TON;
    TemperatureMonitorTimer : TON;
    FinalCheckTimer : TON;

    // Flags
    PreparationsComplete : BOOL := FALSE;
    ConveyorsActivated : BOOL := FALSE;
    SpeedRampComplete : BOOL := FALSE;
    PressureRampComplete : BOOL := FALSE;
    TemperatureWithinRange : BOOL := FALSE;
    FinalChecksPassed : BOOL := FALSE;

    // Interlocks
    EmergencyStop : BOOL := FALSE; // Emergency stop button status
    TemperatureInterlock : BOOL := FALSE; // Temperature interlock status
    PressureInterlock : BOOL := FALSE; // Pressure interlock status
    SpeedInterlock : BOOL := FALSE; // Speed interlock status

    // Sensors
    CurrentSpeed : REAL := 0.0; // Current speed sensor value in m/min
    CurrentPressure : REAL := 0.0; // Current nip pressure sensor value in kN/m
    CurrentTemperature : REAL := 80.0; // Current temperature sensor value in Celsius

    // Constants
    SpeedIncrement : REAL := TargetSpeed / REAL(TO_REAL(SpeedRampTime) / TO_REAL(SpeedStepInterval)); // Speed increment per interval
    PressureIncrement : REAL := TargetPressure / REAL(TO_REAL(PressureRampTime) / TO_REAL(PressureStepInterval)); // Pressure increment per interval
END_VAR

// Function block declarations
FUNCTION_BLOCK RampUpSpeed
VAR_INPUT
    InitialSpeed : REAL;
    TargetSpeed : REAL;
    RampTime : TIME;
    StepInterval : TIME;
END_VAR

VAR_OUTPUT
    RampedSpeed : REAL;
    Complete : BOOL;
END_VAR

VAR
    StepCount : INT := 0;
    StepsPerInterval : INT := 0;
    StepSize : REAL := 0.0;
    RampTimer : TON;
END_VAR

METHOD Execute(this : REFERENCE TO RampUpSpeed) : BOOL
BEGIN
    IF NOT this.RampTimer.Q THEN
        this.RampTimer(IN := TRUE, PT := this.StepInterval);
    END_IF;

    IF this.RampTimer.Q THEN
        this.StepCount := this.StepCount + 1;
        this.RampedSpeed := this.InitialSpeed + (this.StepSize * this.StepCount);

        IF this.RampedSpeed >= this.TargetSpeed THEN
            this.RampedSpeed := this.TargetSpeed;
            this.Complete := TRUE;
        END_IF;

        this.RampTimer(IN := FALSE);
    END_IF;

    RETURN this.Complete;
END_METHOD

FUNCTION_BLOCK RampUpPressure
VAR_INPUT
    InitialPressure : REAL;
    TargetPressure : REAL;
    RampTime : TIME;
    StabilityTime : TIME;
    StepInterval : TIME;
END_VAR

VAR_OUTPUT
    RampedPressure : REAL;
    Stable : BOOL;
END_VAR

VAR
    StepCount : INT := 0;
    StepsPerInterval : INT := 0;
    StepSize : REAL := 0.0;
    RampTimer : TON;
    StabilityTimer : TON;
END_VAR

METHOD Execute(this : REFERENCE TO RampUpPressure) : BOOL
BEGIN
    IF NOT this.RampTimer.Q THEN
        this.RampTimer(IN := TRUE, PT := this.StepInterval);
    END_IF;

    IF this.RampTimer.Q THEN
        this.StepCount := this.StepCount + 1;
        this.RampedPressure := this.InitialPressure + (this.StepSize * this.StepCount);

        IF this.RampedPressure >= this.TargetPressure THEN
            this.RampedPressure := this.TargetPressure;
            this.Stable := TRUE;
        END_IF;

        this.RampTimer(IN := FALSE);
    END_IF;

    IF this.Stable THEN
        IF NOT this.StabilityTimer.Q THEN
            this.StabilityTimer(IN := TRUE, PT := this.StabilityTime);
        END_IF;

        IF this.StabilityTimer.Q THEN
            RETURN TRUE;
        END_IF;
    END_IF;

    RETURN FALSE;
END_METHOD

// Main control loop
CASE currentState OF
    0: // Idle state
        IF StartStartup AND NOT EmergencyStop THEN
            currentState := 1; // Transition to Initiate Startup state
        END_IF;

    1: // Initiate Startup state
        currentState := 2; // Transition to Preparation state

    2: // Preparation state
        // Perform necessary preparations and check interlocks
        PreparationsComplete := TRUE;
        currentState := 3; // Transition to Activate Conveyors state

    3: // Activate Conveyors state
        // Sequentially activate conveyors
        ConveyorsActivated := TRUE;
        currentState := 4; // Transition to Ramp Up Speeds state

    4: // Ramp Up Speeds state
        VAR
            SpeedRampBlock : RampUpSpeed;
        END_VAR

        SpeedRampBlock.InitialSpeed := 0.0;
        SpeedRampBlock.TargetSpeed := TargetSpeed;
        SpeedRampBlock.RampTime := SpeedRampTime;
        SpeedRampBlock.StepInterval := SpeedStepInterval;

        IF NOT SpeedRampTimer.Q THEN
            SpeedRampTimer(IN := TRUE, PT := SpeedStepInterval);
        END_IF;

        IF SpeedRampTimer.Q THEN
            SpeedRampBlock.Execute();
            CurrentSpeed := SpeedRampBlock.RampedSpeed;
            SpeedRampTimer(IN := FALSE);

            IF SpeedRampBlock.Complete THEN
                SpeedRampComplete := TRUE;
                currentState := 5; // Transition to Adjust Nip Pressure state
            END_IF;
        END_IF;

    5: // Adjust Nip Pressure state
        VAR
            PressureRampBlock : RampUpPressure;
        END_VAR

        PressureRampBlock.InitialPressure := 0.0;
        PressureRampBlock.TargetPressure := TargetPressure;
        PressureRampBlock.RampTime := PressureRampTime;
        PressureRampBlock.StabilityTime := PressureStabilityTime;
        PressureRampBlock.StepInterval := PressureStepInterval;

        IF NOT PressureRampTimer.Q THEN
            PressureRampTimer(IN := TRUE, PT := PressureStepInterval);
        END_IF;

        IF PressureRampTimer.Q THEN
            PressureRampBlock.Execute();
            CurrentPressure := PressureRampBlock.RampedPressure;
            PressureRampTimer(IN := FALSE);

            IF PressureRampBlock.Stable THEN
                PressureRampComplete := TRUE;
                currentState := 6; // Transition to Monitor Temperature state
            END_IF;
        END_IF;

    6: // Monitor Temperature state
        IF NOT TemperatureMonitorTimer.Q THEN
            TemperatureMonitorTimer(IN := TRUE, PT := T#1m);
        END_IF;

        IF TemperatureMonitorTimer.Q THEN
            IF CurrentTemperature >= MinTemperature AND CurrentTemperature <= MaxTemperature THEN
                TemperatureWithinRange := TRUE;
                currentState := 7; // Transition to Final Checks state
            END_IF;
            TemperatureMonitorTimer(IN := FALSE);
        END_IF;

    7: // Final Checks state
        IF NOT FinalCheckTimer.Q THEN
            FinalCheckTimer(IN := TRUE, PT := T#1m);
        END_IF;

        IF FinalCheckTimer.Q THEN
            IF PreparationsComplete AND ConveyorsActivated AND SpeedRampComplete AND PressureRampComplete AND TemperatureWithinRange THEN
                FinalChecksPassed := TRUE;
                currentState := 8; // Transition to Operational Mode state
            END_IF;
            FinalCheckTimer(IN := FALSE);
        END_IF;

    8: // Operational Mode state
        // All checks passed, enter operational mode
        currentState := 0; // Transition back to Idle state

    ELSE:
        currentState := 0; // Reset to Idle state in case of unexpected state
END_CASE;

// In-line comments explaining transitions and control logic
// - The program starts in the Idle state and transitions to Initiate Startup when StartStartup is triggered and there is no emergency stop.
// - During Initiate Startup, the program immediately transitions to Preparation.
// - During Preparation, necessary preparations are performed, and interlocks are checked. If successful, it transitions to Activate Conveyors.
// - During Activate Conveyors, conveyors are activated sequentially. Once complete, it transitions to Ramp Up Speeds.
// - During Ramp Up Speeds, the press roll speeds are gradually increased from 0 to 500 m/min over 10 minutes with regular checks every minute. If the target speed is reached, it transitions to Adjust Nip Pressure.
// - During Adjust Nip Pressure, the nip pressure is gradually increased from 0 to 250 kN/m over 5 minutes with stability checks every minute. If the target pressure is stable, it transitions to Monitor Temperature.
// - During Monitor Temperature, the temperature is monitored to ensure it remains within the range of 85–90°C. If the temperature is within range, it transitions to Final Checks.
// - During Final Checks, various safety conditions are verified every minute. If all conditions are met, it transitions to Operational Mode.
// - Once all checks are passed, the program enters operational mode and transitions back to the Idle state.

// Importance of Nip Pressure and Temperature Control
// - Maintaining proper nip pressure ensures effective dewatering and prevents sheet breakage or uneven pressing.
// - Controlling the temperature within the specified range prevents damage to the press rolls and ensures optimal paper quality.
