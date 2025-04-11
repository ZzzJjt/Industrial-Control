```
PROGRAM PRESSURE_RELIEF_SYSTEM
VAR
    // Inputs
    PT_101: REAL; // Pressure Sensor Reading (psi)
    RELIEF_VALVE_STATUS: BOOL; // Current Relief Valve Status (CLOSED = TRUE, OPEN = FALSE)
    
    // Outputs
    RELIEF_VALVE: BOOL; // Relief Valve Control Signal (OPEN = TRUE)
    EMERGENCY_STOP: BOOL; // Emergency Stop Status
    SENSOR_FAULT: BOOL; // Sensor Fault Status
    VALVE_FAULT: BOOL; // Valve Fault Status
    
    // Constants
    HIGH_PRESSURE_LIMIT: REAL := 150.0; // psi
    SAFE_THRESHOLD: REAL := 120.0; // psi
BEGIN
    // Initialize Emergency Stop Status and Fault Flags
    EMERGENCY_STOP := FALSE;
    SENSOR_FAULT := FALSE;
    VALVE_FAULT := FALSE;

    // Check for High Pressure Condition
    IF PT_101 >= HIGH_PRESSURE_LIMIT THEN
        EMERGENCY_STOP := TRUE;
        RELIEF_VALVE := TRUE; // Open Relief Valve
    END_IF;

    // Check for Pressure Below Safe Threshold
    IF PT_101 <= SAFE_THRESHOLD THEN
        EMERGENCY_STOP := FALSE;
        RELIEF_VALVE := FALSE; // Close Relief Valve
    END_IF;

    // Fail-Safe Mechanism Handling
    // Assume SENSOR_FAULT is set externally if sensor readings are erratic or outside valid range
    IF SENSOR_FAULT THEN
        EMERGENCY_STOP := TRUE;
        RELIEF_VALVE := TRUE; // Open Relief Valve if sensor fails
    END_IF;

    // Assume VALVE_FAULT is set externally if valve does not respond to control signal
    IF VALVE_FAULT THEN
        EMERGENCY_STOP := TRUE;
        RELIEF_VALVE := TRUE; // Force valve open if valve malfunctions
    END_IF;

    // Manual Reset Logic (for demonstration purposes, assume manual reset via flag)
    IF NOT EMERGENCY_STOP THEN
        // Reset flags if the pressure is back to safe threshold and no faults are present
        SENSOR_FAULT := FALSE;
        VALVE_FAULT := FALSE;
    END_IF;
END
END_PROGRAM
```
