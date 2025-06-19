PROGRAM PressSectionStartupControl
VAR
    // Inputs
    StartButton : BOOL; // Start button input
    StopButton : BOOL;  // Stop button input
    EmergencyStop : BOOL; // Emergency stop input
    SafetyInterlockClear : BOOL := TRUE; // Safety interlock status

    // Outputs
    ConveyorAActive : BOOL; // Conveyor A output
    ConveyorBActive : BOOL; // Conveyor B output
    ConveyorCActive : BOOL; // Conveyor C output
    Roll1Active : BOOL; // Roll 1 output
    Roll2Active : BOOL; // Roll 2 output
    Roll3Active : BOOL; // Roll 3 output
    RollSpeedCommand : REAL; // Commanded speed for press rolls (m/min)
    NipPressureCommand : REAL; // Commanded nip pressure (kN/m)
    HeaterOutput : BOOL; // Heater output
    CoolerOutput : BOOL; // Cooler output

    // Internal Variables
    CurrentPhase : INT := 0; // Phase tracker: 0 = Idle, 1 = Safety Checks, 2 = Conveyors, 3 = Rolls, 4 = Speed Ramp-Up, 5 = Pressure Ramp-Up, 6 = Temperature Regulation, 7 = Synchronization
    Running : BOOL := FALSE; // Machine running status
    EmergencyStopped : BOOL := FALSE; // Emergency stop status

    // Process Parameters
    TargetRollSpeed : REAL := 500.0; // Target speed for press rolls (m/min)
    SpeedRampDuration : TIME := T#5m; // Duration for speed ramp up
    TargetNipPressure : REAL := 250.0; // Target nip pressure (kN/m)
    PressureRampDuration : TIME := T#5m; // Duration for pressure ramp up
    TargetTemperatureMin : REAL := 85.0; // Minimum target temperature (°C)
    TargetTemperatureMax : REAL := 90.0; // Maximum target temperature (°C)

    // Timers
    TimerConveyors : TON; // Timer for conveyor activation
    TimerRolls : TON; // Timer for roll activation
    TimerSpeedRamp : TON; // Timer for speed ramp up
    TimerPressureRamp : TON; // Timer for pressure ramp up
    TimerTemperatureRegulation : TON; // Timer for temperature regulation

    // Sensors
    CurrentRollSpeed : REAL := 0.0; // Current speed of press rolls (m/min)
    CurrentNipPressure : REAL := 0.0; // Current nip pressure (kN/m)
    CurrentTemperature : REAL := 0.0; // Current temperature reading from sensor

    // Methods
    PROCEDURE PerformSafetyChecks();
    PROCEDURE ActivateConveyors();
    PROCEDURE ActivateRolls();
    PROCEDURE RampUpRollSpeed();
    PROCEDURE RampUpNipPressure();
    PROCEDURE RegulateTemperature();
    PROCEDURE SynchronizeMotion();
END_VAR

// Initialize variables
ConveyorAActive := FALSE;
ConveyorBActive := FALSE;
ConveyorCActive := FALSE;
Roll1Active := FALSE;
Roll2Active := FALSE;
Roll3Active := FALSE;
RollSpeedCommand := 0.0;
NipPressureCommand := 0.0;
HeaterOutput := FALSE;
CoolerOutput := FALSE;

// Main control loop
IF EmergencyStop THEN
    EmergencyStopped := TRUE;
    Running := FALSE;
    CurrentPhase := 0; // Reset phase
ELSE
    IF StartButton AND NOT Running AND NOT EmergencyStopped THEN
        Running := TRUE;
        CurrentPhase := 1; // Initiate Safety Checks phase
    ELSIF StopButton AND Running THEN
        Running := FALSE;
        CurrentPhase := 0; // Reset phase
    END_IF;
END_IF;

// Phase Control Logic
IF Running THEN
    CASE CurrentPhase OF
        1: // Safety Checks phase
            PerformSafetyChecks();
            IF SafetyInterlockClear THEN
                CurrentPhase := 2; // Transition to Conveyors phase
            END_IF;
        2: // Conveyors phase
            ActivateConveyors();
            IF TimerConveyors.Q THEN
                CurrentPhase := 3; // Transition to Rolls phase
            END_IF;
        3: // Rolls phase
            ActivateRolls();
            IF TimerRolls.Q THEN
                CurrentPhase := 4; // Transition to Speed Ramp-Up phase
            END_IF;
        4: // Speed Ramp-Up phase
            RampUpRollSpeed();
            IF TimerSpeedRamp.Q THEN
                CurrentPhase := 5; // Transition to Pressure Ramp-Up phase
            END_IF;
        5: // Pressure Ramp-Up phase
            RampUpNipPressure();
            IF TimerPressureRamp.Q THEN
                CurrentPhase := 6; // Transition to Temperature Regulation phase
            END_IF;
        6: // Temperature Regulation phase
            RegulateTemperature();
            IF CurrentTemperature >= TargetTemperatureMin AND CurrentTemperature <= TargetTemperatureMax THEN
                CurrentPhase := 7; // Transition to Synchronization phase
            END_IF;
        7: // Synchronization phase
            SynchronizeMotion();
            IF CurrentRollSpeed = TargetRollSpeed AND CurrentNipPressure = TargetNipPressure THEN
                Running := FALSE; // End of startup process
                CurrentPhase := 0; // Reset phase
            END_IF;
        ELSE
            CurrentPhase := 0; // Reset phase
    END_CASE;
END_IF;

// Emergency Stop Handling
IF EmergencyStopped THEN
    // Ensure all components are deactivated
    ConveyorAActive := FALSE;
    ConveyorBActive := FALSE;
    ConveyorCActive := FALSE;
    Roll1Active := FALSE;
    Roll2Active := FALSE;
    Roll3Active := FALSE;
    RollSpeedCommand := 0.0;
    NipPressureCommand := 0.0;
    HeaterOutput := FALSE;
    CoolerOutput := FALSE;
    Running := FALSE;
    EmergencyStopped := FALSE; // Reset emergency stop flag after handling
END_IF;

// Modular Methods
PROCEDURE PerformSafetyChecks()
    // Check safety interlocks
    IF SafetyInterlockClear THEN
        // Proceed with startup
    ELSE
        // Stop and wait for safety issues to be resolved
        Running := FALSE;
        CurrentPhase := 0; // Reset phase
    END_IF;
END_PROCEDURE

PROCEDURE ActivateConveyors()
    TimerConveyors(IN := TRUE, PT := T#1s); // Simulated delay for conveyor activation
    IF TimerConveyors.IN_TIME > T#0s THEN
        ConveyorAActive := TRUE;
        TimerConveyors(IN := TRUE, PT := T#1s); // Delay for next conveyor
        IF TimerConveyors.IN_TIME > T#1s THEN
            ConveyorBActive := TRUE;
            TimerConveyors(IN := TRUE, PT := T#1s); // Delay for next conveyor
            IF TimerConveyors.IN_TIME > T#2s THEN
                ConveyorCActive := TRUE;
            END_IF;
        END_IF;
    END_IF;
END_PROCEDURE

PROCEDURE ActivateRolls()
    TimerRolls(IN := TRUE, PT := T#1s); // Simulated delay for roll activation
    IF TimerRolls.IN_TIME > T#0s THEN
        Roll1Active := TRUE;
        TimerRolls(IN := TRUE, PT := T#1s); // Delay for next roll
        IF TimerRolls.IN_TIME > T#1s THEN
            Roll2Active := TRUE;
            TimerRolls(IN := TRUE, PT := T#1s); // Delay for next roll
            IF TimerRolls.IN_TIME > T#2s THEN
                Roll3Active := TRUE;
            END_IF;
        END_IF;
    END_IF;
END_PROCEDURE

PROCEDURE RampUpRollSpeed()
    TimerSpeedRamp(IN := TRUE, PT := SpeedRampDuration);
    // Linear ramp-up of roll speed
    IF TimerSpeedRamp.IN_TIME > T#0s THEN
        RollSpeedCommand := (TimerSpeedRamp.ET / SpeedRampDuration) * TargetRollSpeed;
        CurrentRollSpeed := RollSpeedCommand; // Simulate speed change
    END_IF;
END_PROCEDURE

PROCEDURE RampUpNipPressure()
    TimerPressureRamp(IN := TRUE, PT := PressureRampDuration);
    // Linear ramp-up of nip pressure
    IF TimerPressureRamp.IN_TIME > T#0s THEN
        NipPressureCommand := (TimerPressureRamp.ET / PressureRampDuration) * TargetNipPressure;
        CurrentNipPressure := NipPressureCommand; // Simulate pressure change
    END_IF;
END_PROCEDURE

PROCEDURE RegulateTemperature()
    // PID control for temperature regulation
    IF CurrentTemperature < TargetTemperatureMin THEN
        HeaterOutput := TRUE;
        CoolerOutput := FALSE;
        CurrentTemperature := CurrentTemperature + 1.0; // Simulate temperature increase
    ELSIF CurrentTemperature > TargetTemperatureMax THEN
        HeaterOutput := FALSE;
        CoolerOutput := TRUE;
        CurrentTemperature := CurrentTemperature - 1.0; // Simulate temperature decrease
    ELSE
        HeaterOutput := FALSE;
        CoolerOutput := FALSE;
    END_IF;
END_PROCEDURE

PROCEDURE SynchronizeMotion()
    // Ensure synchronized motion between rolls and conveyors
    // This can involve more complex logic depending on the specific requirements
    // For simplicity, we assume that once the target speed and pressure are reached, synchronization is achieved
END_PROCEDURE



