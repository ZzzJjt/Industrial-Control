FUNCTION_BLOCK OverfillProtectionInterlock
VAR_INPUT
    LevelSensor : REAL; // Liquid level sensor reading as percentage of full scale
    SensorValid : BOOL; // Validity status of the level sensor
    ValveResponse : BOOL; // Status indicating if the inlet valve responded correctly
END_VAR

VAR_OUTPUT
    InletValve_Close : BOOL; // Command to close the inlet valve
    ShutdownLatch : BOOL; // Latched shutdown state
END_VAR

VAR
    HighLevelSetpoint : REAL := 95.0; // High-level setpoint as percentage of full scale
    LowResetThreshold : REAL := 85.0; // Lower reset threshold as percentage of full scale
    SensorFailureDetected : BOOL;
    ValveMalfunctionDetected : BOOL;
END_VAR

// Main execution logic
InletValve_Close := FALSE;
ShutdownLatch := FALSE;

// Check for sensor validity
IF NOT SensorValid THEN
    SensorFailureDetected := TRUE;
ELSE
    SensorFailureDetected := FALSE;
END_IF;

// Check for valve response
IF NOT ValveResponse THEN
    ValveMalfunctionDetected := TRUE;
ELSE
    ValveMalfunctionDetected := FALSE;
END_IF;

// Determine if shutdown latch should be activated
IF (LevelSensor > HighLevelSetpoint) OR SensorFailureDetected OR ValveMalfunctionDetected THEN
    ShutdownLatch := TRUE;
END_IF;

// Close inlet valve if shutdown latch is active
IF ShutdownLatch THEN
    InletValve_Close := TRUE;
END_IF;

// Clear shutdown latch if level drops below low reset threshold and no faults detected
IF (LevelSensor < LowResetThreshold) AND NOT SensorFailureDetected AND NOT ValveMalfunctionDetected THEN
    ShutdownLatch := FALSE;
END_IF;



