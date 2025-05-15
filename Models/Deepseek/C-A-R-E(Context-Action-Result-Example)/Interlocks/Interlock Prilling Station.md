FUNCTION_BLOCK PrillingStationInterlocks
VAR_INPUT
    TowerTemperature : REAL; // Temperature in the prill tower in °C
    MeltSystemPressure : REAL; // Pressure in the melt system in bar
    CoolingAirFlow : REAL; // Cooling air flow in m³/min
    ProductLevel : REAL; // Prill accumulation level as percentage of bin capacity
    MeltFlow : BOOL; // Status indicating if melt flow is present
    ScrubberStatus : BOOL; // Status indicating if scrubber is functioning correctly
    EmergencyStopButton : BOOL; // Manual emergency stop button state
    InertGasFlow : REAL; // Inert gas flow as percentage of required flow
END_VAR

VAR_OUTPUT
    TowerShutdown : BOOL; // Command to shut down the prill tower
    PumpShutdown : BOOL; // Command to shut down pumps
    ReliefValvesOpen : BOOL; // Command to open relief valves
    MeltFeedHalt : BOOL; // Command to halt melt feed
    PrillingHeadDisable : BOOL; // Command to disable prilling head
    EmissionsSourceStop : BOOL; // Command to stop emissions source
    SystemShutdown : BOOL; // Command to shut down entire system
    ProcessesStop : BOOL; // Command to stop all processes relying on inert gas
    AlarmTriggered : BOOL; // Alarm triggered due to any interlock activation
END_VAR

VAR
    OverTempLatch : BOOL;
    HighPressureLatch : BOOL;
    LowCoolingAirLatch : BOOL;
    HighProductLevelLatch : BOOL;
    PumpFeederFailureLatch : BOOL;
    ScrubberFailureLatch : BOOL;
    EmergencyStopLatch : BOOL;
    InertGasLossLatch : BOOL;
END_VAR

// Constants for process limits
CONSTANT
    TOWER_TEMP_LIMIT_HIGH : REAL := 250.0; // Overtemperature limit in °C
    TOWER_TEMP_LIMIT_LOW : REAL := 240.0; // Safe temperature threshold in °C
    MELT_SYSTEM_PRESSURE_LIMIT_HIGH : REAL := 12.0; // High pressure limit in bar
    MELT_SYSTEM_PRESSURE_LIMIT_LOW : REAL := 10.0; // Safe pressure threshold in bar
    COOLING_AIR_FLOW_MIN : REAL := 100.0; // Minimum cooling air flow in m³/min
    COOLING_AIR_FLOW_VERIFY : REAL := 110.0; // Verified cooling air flow in m³/min
    PRODUCT_LEVEL_HIGH : REAL := 90.0; // High product level threshold in %
    PRODUCT_LEVEL_LOW : REAL := 70.0; // Safe product level threshold in %
    INERT_GAS_FLOW_MIN : REAL := 90.0; // Minimum inert gas flow in %
    INERT_GAS_FLOW_VERIFY : REAL := 95.0; // Verified inert gas flow in %
END_CONSTANT

// Main execution logic
TowerShutdown := FALSE;
PumpShutdown := FALSE;
ReliefValvesOpen := FALSE;
MeltFeedHalt := FALSE;
PrillingHeadDisable := FALSE;
EmissionsSourceStop := FALSE;
SystemShutdown := FALSE;
ProcessesStop := FALSE;
AlarmTriggered := FALSE;

// Overtemperature Interlock
IF TowerTemperature > TOWER_TEMP_LIMIT_HIGH THEN
    OverTempLatch := TRUE;
END_IF;

IF OverTempLatch THEN
    TowerShutdown := TRUE;
    AlarmTriggered := TRUE;
END_IF;

IF TowerTemperature < TOWER_TEMP_LIMIT_LOW THEN
    OverTempLatch := FALSE;
END_IF;

// High Pressure Interlock
IF MeltSystemPressure > MELT_SYSTEM_PRESSURE_LIMIT_HIGH THEN
    HighPressureLatch := TRUE;
END_IF;

IF HighPressureLatch THEN
    ReliefValvesOpen := TRUE;
    PumpShutdown := TRUE;
    AlarmTriggered := TRUE;
END_IF;

IF MeltSystemPressure < MELT_SYSTEM_PRESSURE_LIMIT_LOW THEN
    HighPressureLatch := FALSE;
END_IF;

// Low Cooling Air Flow Interlock
IF CoolingAirFlow < COOLING_AIR_FLOW_MIN THEN
    LowCoolingAirLatch := TRUE;
END_IF;

IF LowCoolingAirLatch THEN
    MeltFeedHalt := TRUE;
    PrillingHeadDisable := TRUE;
    AlarmTriggered := TRUE;
END_IF;

IF CoolingAirFlow > COOLING_AIR_FLOW_VERIFY THEN
    LowCoolingAirLatch := FALSE;
END_IF;

// High Product Level Interlock
IF ProductLevel > PRODUCT_LEVEL_HIGH THEN
    HighProductLevelLatch := TRUE;
END_IF;

IF HighProductLevelLatch THEN
    ProcessesStop := TRUE;
    AlarmTriggered := TRUE;
END_IF;

IF ProductLevel < PRODUCT_LEVEL_LOW THEN
    HighProductLevelLatch := FALSE;
END_IF;

// Pump or Feeder Failure Interlock
IF NOT MeltFlow THEN
    PumpFeederFailureLatch := TRUE;
END_IF;

IF PumpFeederFailureLatch THEN
    PrillingHeadDisable := TRUE;
    AlarmTriggered := TRUE;
END_IF;

IF MeltFlow THEN
    PumpFeederFailureLatch := FALSE;
END_IF;

// Scrubber Failure Interlock
IF NOT ScrubberStatus THEN
    ScrubberFailureLatch := TRUE;
END_IF;

IF ScrubberFailureLatch THEN
    EmissionsSourceStop := TRUE;
    AlarmTriggered := TRUE;
END_IF;

IF ScrubberStatus THEN
    ScrubberFailureLatch := FALSE;
END_IF;

// Emergency Stop Interlock
IF EmergencyStopButton THEN
    EmergencyStopLatch := TRUE;
END_IF;

IF EmergencyStopLatch THEN
    SystemShutdown := TRUE;
    AlarmTriggered := TRUE;
END_IF;

// Inerting Gas Loss Interlock
IF InertGasFlow < INERT_GAS_FLOW_MIN THEN
    InertGasLossLatch := TRUE;
END_IF;

IF InertGasLossLatch THEN
    ProcessesStop := TRUE;
    AlarmTriggered := TRUE;
END_IF;

IF InertGasFlow > INERT_GAS_FLOW_VERIFY THEN
    InertGasLossLatch := FALSE;
END_IF;



