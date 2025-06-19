PROGRAM UreaReactionControl
VAR
    // Inputs
    StartButton : BOOL; // Start button input
    StopButton : BOOL;  // Stop button input
    EmergencyStop : BOOL; // Emergency stop input

    // Outputs
    HeaterActive : BOOL; // Heater output
    CoolerActive : BOOL; // Cooler output
    PressureRegulatorActive : BOOL; // Pressure regulator output

    // Internal Variables
    CurrentPhase : INT := 0; // Phase tracker: 0 = Idle, 1 = Heating, 2 = Pressure Regulation, 3 = Cooling
    Running : BOOL := FALSE; // Machine running status
    EmergencyStopped : BOOL := FALSE; // Emergency stop status

    // Process Parameters
    HeatingTargetTemperature : REAL := 180.0; // Target temperature for heating
    PressureSetpoint : REAL := 140.0; // Pressure setpoint for regulation
    ReactionDuration : TIME := T#30m; // Duration for pressure regulation
    CoolingTargetTemperature : REAL := 25.0; // Target temperature for cooling

    // Timers
    TimerHeating : TON; // Timer for heating phase
    TimerPressureRegulation : TON; // Timer for pressure regulation phase
    TimerCooling : TON; // Timer for cooling phase

    // Sensors
    CurrentTemperature : REAL := 0.0; // Current temperature reading from sensor
    CurrentPressure : REAL := 0.0; // Current pressure reading from sensor

    // Methods
    PROCEDURE StartHeating();
    PROCEDURE RegulatePressure();
    PROCEDURE StartCooling();
END_VAR

// Initialize variables
HeaterActive := FALSE;
CoolerActive := FALSE;
PressureRegulatorActive := FALSE;

// Main control loop
IF EmergencyStop THEN
    EmergencyStopped := TRUE;
    Running := FALSE;
    CurrentPhase := 0; // Reset phase
ELSE
    IF StartButton AND NOT Running AND NOT EmergencyStopped THEN
        Running := TRUE;
        CurrentPhase := 1; // Initiate Heating phase
    ELSIF StopButton AND Running THEN
        Running := FALSE;
        CurrentPhase := 0; // Reset phase
    END_IF;
END_IF;

// Phase Control Logic
IF Running THEN
    CASE CurrentPhase OF
        1: // Heating phase
            StartHeating();
            IF CurrentTemperature >= HeatingTargetTemperature THEN
                CurrentPhase := 2; // Transition to Pressure Regulation phase
            END_IF;
        2: // Pressure Regulation phase
            RegulatePressure();
            IF TimerPressureRegulation.Q THEN
                CurrentPhase := 3; // Transition to Cooling phase
            END_IF;
        3: // Cooling phase
            StartCooling();
            IF CurrentTemperature <= CoolingTargetTemperature THEN
                Running := FALSE; // End of process
                CurrentPhase := 0; // Reset phase
            END_IF;
        ELSE
            CurrentPhase := 0; // Reset phase
    END_CASE;
END_IF;

// Emergency Stop Handling
IF EmergencyStopped THEN
    // Ensure all components are deactivated
    HeaterActive := FALSE;
    CoolerActive := FALSE;
    PressureRegulatorActive := FALSE;
    Running := FALSE;
    EmergencyStopped := FALSE; // Reset emergency stop flag after handling
END_IF;

// Modular Methods
PROCEDURE StartHeating()
    HeaterActive := TRUE;
    CoolerActive := FALSE;
    PressureRegulatorActive := FALSE;
    // Simulate temperature increase
    CurrentTemperature := CurrentTemperature + 1.0; // Increment temperature
END_PROCEDURE

PROCEDURE RegulatePressure()
    HeaterActive := FALSE;
    CoolerActive := FALSE;
    PressureRegulatorActive := TRUE;
    TimerPressureRegulation(IN := TRUE, PT := ReactionDuration);
    // Simulate pressure regulation
    IF CurrentPressure < PressureSetpoint THEN
        CurrentPressure := CurrentPressure + 1.0; // Increment pressure
    ELSIF CurrentPressure > PressureSetpoint THEN
        CurrentPressure := CurrentPressure - 1.0; // Decrement pressure
    END_IF;
END_PROCEDURE

PROCEDURE StartCooling()
    HeaterActive := FALSE;
    CoolerActive := TRUE;
    PressureRegulatorActive := FALSE;
    // Simulate temperature decrease
    CurrentTemperature := CurrentTemperature - 1.0; // Decrement temperature
END_PROCEDURE



