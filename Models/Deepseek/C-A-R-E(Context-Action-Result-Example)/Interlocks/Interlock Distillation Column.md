FUNCTION_BLOCK DistillationColumnInterlocks
VAR_INPUT
    PT_101 : REAL; // Pressure transmitter reading in psi
    TT_101 : REAL; // Temperature transmitter reading in °C
    LT_101 : REAL; // Level transmitter reading as percentage of full scale
END_VAR

VAR_OUTPUT
    PRV_101_Open : BOOL; // Command to open PRV-101
    FV_101_Close : BOOL; // Command to close FV-101
    E_101_ShutOff : BOOL; // Command to shut off reboiler heating
END_VAR

// Constants for process limits
CONSTANT
    OVERPRESSURE_LIMIT : REAL := 120.0; // Overpressure limit in psi
    UNDERPRESSURE_LIMIT : REAL := 50.0; // Underpressure limit in psi
    OVERHEATING_LIMIT : REAL := 180.0; // Overheating limit in °C
    LOW_LEVEL_LIMIT : REAL := 20.0; // Low level limit as percentage
    HIGH_LEVEL_LIMIT : REAL := 80.0; // High level limit as percentage
END_CONSTANT

// Main execution logic
PRV_101_Open := FALSE;
FV_101_Close := FALSE;
E_101_ShutOff := FALSE;

// Overpressure protection
IF PT_101 > OVERPRESSURE_LIMIT THEN
    PRV_101_Open := TRUE;
END_IF;

// Underpressure protection
IF PT_101 < UNDERPRESSURE_LIMIT THEN
    FV_101_Close := TRUE;
END_IF;

// Overheating protection
IF TT_101 > OVERHEATING_LIMIT THEN
    E_101_ShutOff := TRUE;
END_IF;

// Low-level protection
IF LT_101 < LOW_LEVEL_LIMIT THEN
    FV_101_Close := TRUE;
END_IF;

// High-level protection
IF LT_101 > HIGH_LEVEL_LIMIT THEN
    PRV_101_Open := TRUE;
END_IF;



