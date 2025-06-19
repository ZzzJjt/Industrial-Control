FUNCTION_BLOCK SubseaWellheadInterlocks
VAR_INPUT
    PT_101 : REAL; // Pressure sensor reading in psi
    TT_101 : REAL; // Temperature sensor reading in °C
    FT_101 : REAL; // Flow rate sensor reading in L/min
    RESET_Button : BOOL; // Manual reset button state
END_VAR

VAR_OUTPUT
    MV_101_Close : BOOL; // Command to close master valve MV-101
    SHUTDOWN : BOOL; // Full shutdown command
END_VAR

VAR
    ShutdownActive : BOOL; // Flag indicating if a shutdown has occurred
END_VAR

// Constants for process limits
CONSTANT
    OVERPRESSURE_LIMIT : REAL := 1500.0; // Overpressure limit in psi
    MIN_FLOW_RATE : REAL := 100.0; // Minimum acceptable flow rate in L/min
    OVERTEMPERATURE_LIMIT : REAL := 120.0; // Overtemperature limit in °C
END_CONSTANT

// Main execution logic
MV_101_Close := FALSE;
SHUTDOWN := FALSE;

IF NOT ShutdownActive THEN
    // Overpressure protection
    IF PT_101 > OVERPRESSURE_LIMIT THEN
        MV_101_Close := TRUE;
        SHUTDOWN := TRUE;
        ShutdownActive := TRUE;
    END_IF;

    // Low flow rate protection
    IF FT_101 < MIN_FLOW_RATE THEN
        MV_101_Close := TRUE;
        SHUTDOWN := TRUE;
        ShutdownActive := TRUE;
    END_IF;

    // Overtemperature protection
    IF TT_101 > OVERTEMPERATURE_LIMIT THEN
        MV_101_Close := TRUE;
        SHUTDOWN := TRUE;
        ShutdownActive := TRUE;
    END_IF;
ELSE
    // Allow manual reset only if all conditions are normal
    IF RESET_Button AND (PT_101 <= OVERPRESSURE_LIMIT) AND (FT_101 >= MIN_FLOW_RATE) AND (TT_101 <= OVERTEMPERATURE_LIMIT) THEN
        ShutdownActive := FALSE;
    END_IF;
END_IF;



