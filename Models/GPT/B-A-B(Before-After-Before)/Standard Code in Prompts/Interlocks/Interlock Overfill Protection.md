```
PROGRAM OVERFILL_PROTECTION_SYSTEM
VAR
    // Inputs
    LS_101: REAL; // Level Sensor Reading
    VALVE_STATUS: BOOL; // Current Valve Status (OPEN = TRUE, CLOSED = FALSE)

    // Outputs
    INLET_VALVE: BOOL; // Inlet Valve Control Signal (CLOSE = TRUE)
    EMERGENCY_STOP: BOOL; // Emergency Stop Status
    SENSOR_FAULT: BOOL; // Sensor Fault Status
    VALVE_FAULT: BOOL; // Valve Fault Status
    
    // Constants
    HIGH_LEVEL_SETPOINT: REAL := 90.0; // Percentage of tank capacity
    SAFE_THRESHOLD: REAL := 85.0; // Percentage of tank capacity
BEGIN
    // Initialize Emergency Stop Status and Fault Flags
    EMERGENCY_STOP := FALSE;
    SENSOR_FAULT := FALSE;
    VALVE_FAULT := FALSE;

    // Check for High Level Condition
    IF LS_101 >= HIGH_LEVEL_SETPOINT THEN
        EMERGENCY_STOP := TRUE;
        INLET_VALVE := TRUE; // Close Inlet Valve
    END_IF;

    // Check for Level Below Safe Threshold
    IF LS_101 <= SAFE_THRESHOLD THEN
        EMERGENCY_STOP := FALSE;
        INLET_VALVE := FALSE; // Open Inlet Valve
    END_IF;

    // Fail-Safe Mechanism Handling
    // Assume SENSOR_FAULT is set externally if sensor readings are erratic or outside valid range
    IF SENSOR_FAULT THEN
        EMERGENCY_STOP := TRUE;
        INLET_VALVE := TRUE; // Close Inlet Valve if sensor fails
    END_IF;

    // Assume VALVE_FAULT is set externally if valve does not respond to control signal
    IF VALVE_FAULT THEN
        EMERGENCY_STOP := TRUE;
        INLET_VALVE := TRUE; // Force valve closed if valve malfunctions
    END_IF;

    // Manual Reset Logic (for demonstration purposes, assume manual reset via flag)
    IF NOT EMERGENCY_STOP THEN
        // Reset flags if the level is back to safe threshold and no faults are present
        SENSOR_FAULT := FALSE;
        VALVE_FAULT := FALSE;
    END_IF;
END
END_PROGRAM
```
