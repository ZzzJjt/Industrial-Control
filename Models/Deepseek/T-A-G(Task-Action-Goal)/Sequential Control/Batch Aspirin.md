PROGRAM AspirinProductionControl
VAR
    // Inputs
    StartButton : BOOL; // Start button input
    StopButton : BOOL;  // Stop button input
    EmergencyStop : BOOL; // Emergency stop input

    // Outputs
    HeaterActive : BOOL; // Heater output
    MixerActive : BOOL; // Mixer output
    PressureValveOpen : BOOL; // Pressure valve output

    // Internal Variables
    CurrentPhase : INT := 0; // Phase tracker: 0 = Idle, 1 = Heating, 2 = Mixing, 3 = Holding
    Running : BOOL := FALSE; // Machine running status
    EmergencyStopped : BOOL := FALSE; // Emergency stop status

    // Process Parameters
    TargetTemperature : REAL := 75.0; // Target temperature for heating
    TargetPressure : REAL := 1.5; // Target pressure for holding
    MixingDuration : TIME := T#10m; // Duration for mixing
    HoldingDuration : TIME := T#30m; // Duration for holding

    // Timers
    TimerHeating : TON; // Timer for heating phase
    TimerMixing : TON; // Timer for mixing phase
    TimerHolding : TON; // Timer for holding phase

    // Sensors
    CurrentTemperature : REAL := 0.0; // Current temperature reading from sensor
    CurrentPressure : REAL := 0.0; // Current pressure reading from sensor

    // Methods
    PROCEDURE StartHeating();
    PROCEDURE StartMixing();
    PROCEDURE HoldReaction();
END_VAR

// Initialize variables
HeaterActive := FALSE;
MixerActive := FALSE;
PressureValveOpen := FALSE;

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
            IF CurrentTemperature >= TargetTemperature THEN
                CurrentPhase := 2; // Transition to Mixing phase
            END_IF;
        2: // Mixing phase
            StartMixing();
            IF TimerMixing.Q THEN
                CurrentPhase := 3; // Transition to Holding phase
            END_IF;
        3: // Holding phase
            HoldReaction();
            IF TimerHolding.Q THEN
                Running := FALSE; // End of Reaction step
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
    MixerActive := FALSE;
    PressureValveOpen := FALSE;
    Running := FALSE;
    EmergencyStopped := FALSE; // Reset emergency stop flag after handling
END_IF;

// Modular Methods
PROCEDURE StartHeating()
    HeaterActive := TRUE;
    // Simulate temperature increase
    CurrentTemperature := CurrentTemperature + 1.0; // Increment temperature
END_PROCEDURE

PROCEDURE StartMixing()
    MixerActive := TRUE;
    TimerMixing(IN := TRUE, PT := MixingDuration);
END_PROCEDURE

PROCEDURE HoldReaction()
    PressureValveOpen := TRUE;
    TimerHolding(IN := TRUE, PT := HoldingDuration);
    // Simulate pressure adjustment
    CurrentPressure := CurrentPressure + 0.1; // Increment pressure
END_PROCEDURE

// Crystallization Phase Control Program
VAR
    CrystallizationRunning : BOOL := FALSE;
    CrystallizationPhase : INT := 0; // Phase tracker: 0 = Idle, 1 = Cooling, 2 = Seeding, 3 = Agitation
    CrystallizationTimer : TON; // Timer for crystallization phases
    CrystallizationTargetTemp : REAL := 50.0; // Target temperature for cooling
END_VAR

// Crystallization Control Logic
IF CrystallizationRunning THEN
    CASE CrystallizationPhase OF
        1: // Cooling phase
            HeaterActive := FALSE;
            // Simulate temperature decrease
            CurrentTemperature := CurrentTemperature - 1.0; // Decrement temperature
            IF CurrentTemperature <= CrystallizationTargetTemp THEN
                CrystallizationPhase := 2; // Transition to Seeding phase
            END_IF;
        2: // Seeding phase
            // Add seed crystals
            CrystallizationPhase := 3; // Transition to Agitation phase
        3: // Agitation phase
            MixerActive := TRUE;
            CrystallizationTimer(IN := TRUE, PT := T#2h); // Agitate for 2 hours
            IF CrystallizationTimer.Q THEN
                CrystallizationRunning := FALSE; // End of Crystallization step
                CrystallizationPhase := 0; // Reset phase
            END_IF;
        ELSE
            CrystallizationPhase := 0; // Reset phase
    END_CASE;
END_IF;

// Drying Phase Control Program
VAR
    DryingRunning : BOOL := FALSE;
    DryingPhase : INT := 0; // Phase tracker: 0 = Idle, 1 = Heating, 2 = Holding
    DryingTimer : TON; // Timer for drying phases
    DryingTargetTemp : REAL := 90.0; // Target temperature for drying
END_VAR

// Drying Control Logic
IF DryingRunning THEN
    CASE DryingPhase OF
        1: // Heating phase
            HeaterActive := TRUE;
            // Simulate temperature increase
            CurrentTemperature := CurrentTemperature + 1.0; // Increment temperature
            IF CurrentTemperature >= DryingTargetTemp THEN
                DryingPhase := 2; // Transition to Holding phase
            END_IF;
        2: // Holding phase
            DryingTimer(IN := TRUE, PT := T#4h); // Dry for 4 hours
            IF DryingTimer.Q THEN
                DryingRunning := FALSE; // End of Drying step
                DryingPhase := 0; // Reset phase
            END_IF;
        ELSE
            DryingPhase := 0; // Reset phase
    END_CASE;
END_IF;

// Additional Control Logic for Integration
IF NOT Running AND NOT CrystallizationRunning AND NOT DryingRunning THEN
    IF StartButton AND NOT EmergencyStopped THEN
        Running := TRUE;
        CurrentPhase := 1; // Start Reaction phase
    END_IF;
ELSIF NOT Running AND CrystallizationRunning AND NOT DryingRunning THEN
    IF StartButton AND NOT EmergencyStopped THEN
        CrystallizationRunning := TRUE;
        CrystallizationPhase := 1; // Start Crystallization phase
    END_IF;
ELSIF NOT Running AND NOT CrystallizationRunning AND DryingRunning THEN
    IF StartButton AND NOT EmergencyStopped THEN
        DryingRunning := TRUE;
        DryingPhase := 1; // Start Drying phase
    END_IF;
END_IF;

IF StopButton OR EmergencyStop THEN
    Running := FALSE;
    CrystallizationRunning := FALSE;
    DryingRunning := FALSE;
    CurrentPhase := 0;
    CrystallizationPhase := 0;
    DryingPhase := 0;
END_IF;



