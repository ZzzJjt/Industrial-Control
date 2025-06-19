FUNCTION_BLOCK InterlockEvaluation
VAR_INPUT
    HighPressure : BOOL; // Input signal for high pressure condition
    LowLevel : BOOL; // Input signal for low level condition
    HighTemperature : BOOL; // Input signal for high temperature condition
    LowTemperature : BOOL; // Input signal for low temperature condition
    PressureSensorFault : BOOL; // Input signal for pressure sensor fault
    TemperatureSensorFault : BOOL; // Input signal for temperature sensor fault
    LevelSensorFault : BOOL; // Input signal for level sensor fault
    AgitatorFault : BOOL; // Input signal for agitator fault
    Overheating : BOOL; // Input signal for overheating condition
    Underheating : BOOL; // Input signal for underheating condition
    LevelOverflow : BOOL; // Input signal for level overflow condition
END_VAR

VAR_OUTPUT
    CloseFeedValve : BOOL; // Output signal to close feed valve
    StopFeedPump : BOOL; // Output signal to stop feed pump
    ActivateReliefSystem : BOOL; // Output signal to activate relief system
    IssueHighPressureAlarm : BOOL; // Output signal to issue high-pressure alarm
    CloseProductValve : BOOL; // Output signal to close product valve
    StopCirculationPump : BOOL; // Output signal to stop circulation pump
    IsolateReactor : BOOL; // Output signal to isolate reactor
    IssueLowLevelAlarm : BOOL; // Output signal to issue low-level alarm
    IssueHighTemperatureAlarm : BOOL; // Output signal to issue high-temperature alarm
    IssueLowTemperatureAlarm : BOOL; // Output signal to issue low-temperature alarm
    IssueSensorFaultAlarm : BOOL; // Output signal to issue sensor fault alarm
END_VAR

// Main execution logic
CloseFeedValve := FALSE;
StopFeedPump := FALSE;
ActivateReliefSystem := FALSE;
IssueHighPressureAlarm := FALSE;
CloseProductValve := FALSE;
StopCirculationPump := FALSE;
IsolateReactor := FALSE;
IssueLowLevelAlarm := FALSE;
IssueHighTemperatureAlarm := FALSE;
IssueLowTemperatureAlarm := FALSE;
IssueSensorFaultAlarm := FALSE;

IF HighPressure THEN
    CloseFeedValve := TRUE;
    StopFeedPump := TRUE;
    ActivateReliefSystem := TRUE;
    IssueHighPressureAlarm := TRUE;
END_IF;

IF LowLevel THEN
    CloseProductValve := TRUE;
    StopCirculationPump := TRUE;
    IsolateReactor := TRUE;
    IssueLowLevelAlarm := TRUE;
END_IF;

IF HighTemperature THEN
    StopCirculationPump := TRUE;
    IssueHighTemperatureAlarm := TRUE;
END_IF;

IF LowTemperature THEN
    StopCirculationPump := TRUE;
    IssueLowTemperatureAlarm := TRUE;
END_IF;

IF PressureSensorFault OR TemperatureSensorFault OR LevelSensorFault THEN
    IssueSensorFaultAlarm := TRUE;
END_IF;

IF AgitatorFault THEN
    StopCirculationPump := TRUE;
    IsolateReactor := TRUE;
END_IF;

IF Overheating THEN
    StopCirculationPump := TRUE;
    IssueHighTemperatureAlarm := TRUE;
END_IF;

IF Underheating THEN
    StopCirculationPump := TRUE;
    IssueLowTemperatureAlarm := TRUE;
END_IF;

IF LevelOverflow THEN
    CloseProductValve := TRUE;
    ActivateReliefSystem := TRUE;
    IssueLowLevelAlarm := TRUE;
END_IF;



