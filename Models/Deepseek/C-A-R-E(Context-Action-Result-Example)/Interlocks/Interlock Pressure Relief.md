FUNCTION_BLOCK PressureReliefInterlock
VAR_INPUT
    PT_101 : REAL; // Pressure sensor reading in bar
    SensorValid : BOOL; // Validity status of the pressure sensor
    ValveResponse : BOOL; // Status indicating if the relief valve responded correctly
END_VAR

VAR_OUTPUT
    RELIEF_VALVE_Open : BOOL; // Command to open the relief valve
    SHUTDOWN_LATCH : BOOL; // Latched shutdown state
    ALARM_TRIGGERED : BOOL; // Alarm triggered due to valve malfunction
END_VAR

VAR
    HighPressureLimit : REAL := 15.0; // High pressure limit in bar
    SafeResetThreshold : REAL := 12.0; // Safe reset threshold in bar
    SensorFailureDetected : BOOL;
    ValveMalfunctionDetected : BOOL;
END_VAR

// Main execution logic
RELIEF_VALVE_Open := FALSE;
SHUTDOWN_LATCH := FALSE;
ALARM_TRIGGERED := FALSE;

// Check for sensor validity
IF NOT SensorValid THEN
    SensorFailureDetected := TRUE;
ELSE
    SensorFailureDetected := FALSE;
END_IF;

// Check for valve response
IF NOT ValveResponse THEN
    ValveMalfunctionDetected := TRUE;
    ALARM_TRIGGERED := TRUE;
ELSE
    ValveMalfunctionDetected := FALSE;
    ALARM_TRIGGERED := FALSE;
END_IF;

// Determine if shutdown latch should be activated
IF (PT_101 > HighPressureLimit) OR SensorFailureDetected OR ValveMalfunctionDetected THEN
    SHUTDOWN_LATCH := TRUE;
END_IF;

// Open relief valve if shutdown latch is active
IF SHUTDOWN_LATCH THEN
    RELIEF_VALVE_Open := TRUE;
END_IF;

// Clear shutdown latch if pressure drops below safe reset threshold and no faults detected
IF (PT_101 < SafeResetThreshold) AND NOT SensorFailureDetected AND NOT ValveMalfunctionDetected THEN
    SHUTDOWN_LATCH := FALSE;
END_IF;

// Default to safe state if interlock status is uncertain
IF NOT SensorValid OR NOT ValveResponse THEN
    RELIEF_VALVE_Open := TRUE;
END_IF;



