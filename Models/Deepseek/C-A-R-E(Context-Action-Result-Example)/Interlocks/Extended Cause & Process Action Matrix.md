FUNCTION_BLOCK InterlockEvaluation
VAR_INPUT
    HighPressure : BOOL; // Input signal for high pressure condition
    LowTemperature : BOOL; // Input signal for low temperature condition
    LevelSensorFailure : BOOL; // Input signal for level sensor failure
    AgitatorFault : BOOL; // Input signal for agitator fault
    Overheating : BOOL; // Input signal for overheating condition
    Underheating : BOOL; // Input signal for underheating condition
    PressureSensorFault : BOOL; // Input signal for pressure sensor fault
    TemperatureSensorFault : BOOL; // Input signal for temperature sensor fault
    LevelOverflow : BOOL; // Input signal for level overflow condition
END_VAR

VAR_OUTPUT
    IsolateFeed : BOOL; // Output signal to isolate feed
    ShutDownAgitator : BOOL; // Output signal to shut down agitator
    ActivateReliefValve : BOOL; // Output signal to activate relief valve
    StopHeatingSystem : BOOL; // Output signal to stop heating system
    SafeShutdownSequence : BOOL; // Output signal to initiate safe shutdown sequence
END_VAR

// Main execution logic
IsolateFeed := FALSE;
ShutDownAgitator := FALSE;
ActivateReliefValve := FALSE;
StopHeatingSystem := FALSE;
SafeShutdownSequence := FALSE;

IF HighPressure THEN
    IsolateFeed := TRUE;
    ActivateReliefValve := TRUE;
END_IF;

IF LowTemperature THEN
    ShutDownAgitator := TRUE;
    StopHeatingSystem := TRUE;
END_IF;

IF LevelSensorFailure OR AgitatorFault OR PressureSensorFault OR TemperatureSensorFault THEN
    SafeShutdownSequence := TRUE;
END_IF;

IF Overheating THEN
    StopHeatingSystem := TRUE;
END_IF;

IF Underheating THEN
    ShutDownAgitator := TRUE;
END_IF;

IF LevelOverflow THEN
    ActivateReliefValve := TRUE;
END_IF;



