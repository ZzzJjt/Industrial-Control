PROGRAM OVERFILL_PROTECTION
VAR_INPUT
    Execute : BOOL;          // Starts the interlock checks
    LevelSensor : REAL;      // Liquid level reading from the sensor (e.g., %)
    SensorValid : BOOL;       // Indicates if the sensor signal is valid
    ValveMalfunction : BOOL; // Indicates if there is a valve malfunction
    ManualReset : BOOL;      // Manual reset input to clear the shutdown latch
END_VAR

VAR_OUTPUT
    INLET_VALVE_Open : BOOL; // Status of Inlet Valve (TRUE = Open)
    SHUTDOWN_Latched : BOOL; // Latched shutdown status
    Alarm_Triggered : BOOL;  // Status of Alarm
    Error : BOOL;            // General error flag
    ErrorID : DWORD;         // Detailed error code
END_VAR

VAR
    lastExecute : BOOL := FALSE; // Last state of the Execute input
    lastManualReset : BOOL := FALSE; // Last state of the Manual Reset input
    HIGH_LEVEL_SETPOINT : REAL := 80.0; // High-level setpoint (e.g., 80%)
    RESET_THRESHOLD : REAL := 50.0; // Reset threshold (e.g., 50%)
END_VAR

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        // Reset outputs except latched states
        INLET_VALVE_Open := TRUE;
        Alarm_Triggered := FALSE;
        Error := FALSE;
        ErrorID := 0;

        // Check for sensor validity
        IF NOT SensorValid THEN
            INLET_VALVE_Open := FALSE;
            SHUTDOWN_Latched := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 1; // Invalid sensor signal
        ELSE
            // Check for valve malfunction
            IF ValveMalfunction THEN
                INLET_VALVE_Open := FALSE;
                SHUTDOWN_Latched := TRUE;
                Alarm_Triggered := TRUE;
                Error := TRUE;
                ErrorID := 2; // Valve malfunction detected
            ELSE
                // Check for overfill condition
                IF LevelSensor > HIGH_LEVEL_SETPOINT THEN
                    INLET_VALVE_Open := FALSE;
                    SHUTDOWN_Latched := TRUE;
                    Alarm_Triggered := TRUE;
                    Error := TRUE;
                    ErrorID := 3; // Overfill condition detected
                END_IF;

                // Allow manual reset only if not in error state
                IF NOT Error AND R_TRIG(lastManualReset, ManualReset).Q THEN
                    IF LevelSensor < RESET_THRESHOLD THEN
                        SHUTDOWN_Latched := FALSE;
                    ELSE
                        Alarm_Triggered := TRUE;
                        Error := TRUE;
                        ErrorID := 4; // Cannot reset while overfill condition persists
                    END_IF;
                END_IF;
            END_IF;
        END_IF;
    END_IF;

    lastExecute := Execute;
    lastManualReset := ManualReset;
    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



