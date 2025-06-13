PROGRAM SUBSEA_WELLHEAD_INTERLOCKS
VAR_INPUT
    Execute : BOOL;          // Starts the interlock checks
    PT_101 : REAL;           // Pressure reading from PT sensor (psi)
    TT_101 : REAL;           // Temperature reading from TT sensor (°C)
    FT_101 : REAL;           // Flow rate reading from FT sensor (m³/h)
    RESET : BOOL;            // Manual reset input
END_VAR

VAR_OUTPUT
    MV_101_Closed : BOOL;    // Status of Master Valve (TRUE = Closed)
    SHUTDOWN_Latched : BOOL; // Latched shutdown status
    Alarm_Triggered : BOOL;  // Status of Alarm
    Error : BOOL;            // General error flag
    ErrorID : DWORD;         // Detailed error code
END_VAR

VAR
    lastExecute : BOOL := FALSE; // Last state of the Execute input
    lastRESET : BOOL := FALSE;   // Last state of the RESET input
END_VAR

METHOD Execute : BOOL
BEGIN
    IF R_TRIG(lastExecute, Execute).Q THEN
        // Reset outputs except latched states
        MV_101_Closed := FALSE;
        Alarm_Triggered := FALSE;
        Error := FALSE;
        ErrorID := 0;

        // Check for overpressure
        IF PT_101 > 1500.0 THEN
            MV_101_Closed := TRUE;
            SHUTDOWN_Latched := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 1; // Overpressure detected
        END_IF;

        // Check for low flow rate
        IF FT_101 < 10.0 THEN // Assuming minimum safe flow rate is 10 m³/h
            MV_101_Closed := TRUE;
            SHUTDOWN_Latched := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 2; // Low flow rate detected
        END_IF;

        // Check for overtemperature
        IF TT_101 > 120.0 THEN
            MV_101_Closed := TRUE;
            SHUTDOWN_Latched := TRUE;
            Alarm_Triggered := TRUE;
            Error := TRUE;
            ErrorID := 3; // Overtemperature detected
        END_IF;

        // Allow manual reset only if not in error state
        IF NOT Error AND R_TRIG(lastRESET, RESET).Q THEN
            SHUTDOWN_Latched := FALSE;
        END_IF;
    END_IF;

    lastExecute := Execute;
    lastRESET := RESET;
    RETURN TRUE;
END_METHOD

// Main execution loop
Execute();



