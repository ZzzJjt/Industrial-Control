PROGRAM PRESSURE_RELIEF_INTERLOCK
VAR_INPUT
    Execute : BOOL;          // Starts the interlock checks
    PT_101 : REAL;           // Pressure reading from PT_101 (bar)
    SensorValid : BOOL;       // Indicates if the sensor signal is valid
    ValveMalfunction : BOOL; // Indicates if there is a valve malfunction
    ManualReset : BOOL;      // Manual reset input to clear the shutdown latch
END_VAR

VAR_OUTPUT
    RELIEF_VALVE_Open : BOOL; // Status of Relief Valve (TRUE = Open)
    SHUTDOWN_LATCHED : BOOL;  // Latched shutdown status
    Alarm_Triggered : BOOL;   // Status of Alarm
    Error : BOOL;             // General error flag
    ErrorID : DWORD;          // Detailed error code
END_VAR

VAR
    lastExecute : BOOL := FALSE; // Last state of the Execute input
    lastManualReset : BOOL := FALSE; // Last state of the Manual Reset input
    HIGH_PRESSURE_LIMIT : REAL := 15.0; // High-pressure limit (bar)
    RESET_THRESHOLD : REAL := 12.0; // Reset threshold (bar)
END_VAR

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        // Reset outputs except latched states
        RELIEF_VALVE_Open := FALSE;
        Alarm_Triggered := FALSE;
        Error := FALSE;
        ErrorID := 0;

        // Check for sensor validity
        IF NOT SensorValid THEN
            RELIEF_VALVE_Open := TRUE;
            SHUTDOWN_LATCHED := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 1; // Invalid sensor signal
        ELSE
            // Check for valve malfunction
            IF ValveMalfunction THEN
                RELIEF_VALVE_Open := TRUE;
                SHUTDOWN_LATCHED := TRUE;
                Alarm_Triggered := TRUE;
                Error := TRUE;
                ErrorID := 2; // Valve malfunction detected
            ELSE
                // Check for overpressure condition
                IF PT_101 > HIGH_PRESSURE_LIMIT THEN
                    RELIEF_VALVE_Open := TRUE;
                    SHUTDOWN_LATCHED := TRUE;
                    Alarm_Triggered := TRUE;
                    Error := TRUE;
                    ErrorID := 3; // Overpressure condition detected
                END_IF;

                // Allow manual reset only if not in error state
                IF NOT Error AND R_TRIG(lastManualReset, ManualReset).Q THEN
                    IF PT_101 < RESET_THRESHOLD THEN
                        SHUTDOWN_LATCHED := FALSE;
                    ELSE
                        Alarm_Triggered := TRUE;
                        Error := TRUE;
                        ErrorID := 4; // Cannot reset while overpressure condition persists
                    END_IF;
                END_IF;
            END_IF;
        END_IF;

        // Maintain the relief valve open if latched
        IF SHUTDOWN_LATCHED THEN
            RELIEF_VALVE_Open := TRUE;
        END_IF;
    END_IF;

    lastExecute := Execute;
    lastManualReset := ManualReset;
    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



