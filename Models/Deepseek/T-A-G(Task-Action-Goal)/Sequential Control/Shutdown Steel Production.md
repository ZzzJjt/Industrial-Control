PROGRAM SteelShutdownControl
VAR
    // Inputs
    StartShutdownButton : BOOL; // Start shutdown button input
    StopShutdownButton : BOOL;  // Stop shutdown button input
    EmergencyStop : BOOL; // Emergency stop input

    // Outputs
    HeaterOutput : BOOL; // Heater output
    FuelGasValve : REAL; // Fuel gas valve position (0% to 100%)
    OxygenValve : REAL; // Oxygen valve position (0% to 100%)

    // Internal Variables
    CurrentPhase : INT := 0; // Phase tracker: 0 = Idle, 1 = Temperature Reduction, 2 = Fuel Gas Ramp Down, 3 = Oxygen Regulation
    Running : BOOL := FALSE; // Machine running status
    EmergencyStopped : BOOL := FALSE; // Emergency stop status

    // Process Parameters
    TargetTemperature : REAL := 400.0; // Target temperature for cooling
    InitialFuelFlow : REAL := 100.0; // Initial fuel gas flow percentage
    FinalFuelFlow : REAL := 0.0; // Final fuel gas flow percentage
    RampDuration : TIME := T#12h; // Duration for fuel gas ramp down
    FuelToAirRatio : REAL := 2.5; // Fuel-to-air ratio for oxygen management

    // Timers
    TimerCooling : TON; // Timer for cooling phase
    TimerRampDown : TON; // Timer for fuel gas ramp down phase

    // Sensors
    CurrentTemperature : REAL := 1200.0; // Current temperature reading from sensor
    CurrentFuelFlow : REAL := InitialFuelFlow; // Current fuel gas flow percentage
    CurrentOxygenFlow : REAL := 0.0; // Current oxygen flow percentage

    // Methods
    PROCEDURE ReduceTemperature();
    PROCEDURE RampDownFuelGas();
    PROCEDURE RegulateOxygen();
END_VAR

// Initialize variables
HeaterOutput := TRUE;
FuelGasValve := InitialFuelFlow;
OxygenValve := CalculateOxygenFlow(CurrentFuelFlow);

// Main control loop
IF EmergencyStop THEN
    EmergencyStopped := TRUE;
    Running := FALSE;
    CurrentPhase := 0; // Reset phase
ELSE
    IF StartShutdownButton AND NOT Running AND NOT EmergencyStopped THEN
        Running := TRUE;
        CurrentPhase := 1; // Initiate Temperature Reduction phase
    ELSIF StopShutdownButton AND Running THEN
        Running := FALSE;
        CurrentPhase := 0; // Reset phase
    END_IF;
END_IF;

// Phase Control Logic
IF Running THEN
    CASE CurrentPhase OF
        1: // Temperature Reduction phase
            ReduceTemperature();
            IF CurrentTemperature <= TargetTemperature THEN
                CurrentPhase := 2; // Transition to Fuel Gas Ramp Down phase
            END_IF;
        2: // Fuel Gas Ramp Down phase
            RampDownFuelGas();
            IF TimerRampDown.Q THEN
                CurrentPhase := 3; // Transition to Oxygen Regulation phase
            END_IF;
        3: // Oxygen Regulation phase
            RegulateOxygen();
            IF CurrentFuelFlow <= FinalFuelFlow THEN
                Running := FALSE; // End of shutdown process
                CurrentPhase := 0; // Reset phase
            END_IF;
        ELSE
            CurrentPhase := 0; // Reset phase
    END_CASE;
END_IF;

// Emergency Stop Handling
IF EmergencyStopped THEN
    // Ensure all components are deactivated
    HeaterOutput := FALSE;
    FuelGasValve := 0.0;
    OxygenValve := 0.0;
    Running := FALSE;
    EmergencyStopped := FALSE; // Reset emergency stop flag after handling
END_IF;

// Modular Methods
PROCEDURE ReduceTemperature()
    HeaterOutput := TRUE;
    // Simulate temperature decrease
    CurrentTemperature := CurrentTemperature - 1.0; // Decrement temperature
END_PROCEDURE

PROCEDURE RampDownFuelGas()
    HeaterOutput := FALSE;
    TimerRampDown(IN := TRUE, PT := RampDuration);
    // Gradual reduction of fuel gas flow
    IF TimerRampDown.IN_TIME > T#0s THEN
        CurrentFuelFlow := InitialFuelFlow * (T#12h - TimerRampDown.ET) / T#12h;
        FuelGasValve := CurrentFuelFlow;
    END_IF;
    // Update oxygen flow based on new fuel flow
    CurrentOxygenFlow := CalculateOxygenFlow(CurrentFuelFlow);
    OxygenValve := CurrentOxygenFlow;
END_PROCEDURE

PROCEDURE RegulateOxygen()
    HeaterOutput := FALSE;
    // Further reduce fuel gas flow if necessary
    IF CurrentFuelFlow > FinalFuelFlow THEN
        CurrentFuelFlow := CurrentFuelFlow - 0.1; // Small decrement per cycle
        FuelGasValve := CurrentFuelFlow;
    END_IF;
    // Update oxygen flow based on new fuel flow
    CurrentOxygenFlow := CalculateOxygenFlow(CurrentFuelFlow);
    OxygenValve := CurrentOxygenFlow;
END_PROCEDURE

FUNCTION_BLOCK CalculateOxygenFlow(FuelFlow : REAL) : REAL
VAR_INPUT
    FuelFlow : REAL;
END_VAR
VAR_OUTPUT
    OxygenFlow : REAL;
END_VAR
BEGIN
    OxygenFlow := FuelFlow * FuelToAirRatio;
END_FUNCTION_BLOCK



