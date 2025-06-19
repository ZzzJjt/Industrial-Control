PROGRAM DISTILLATION_COLUMN_INTERLOCKS
VAR_INPUT
    Execute : BOOL;             // Triggers the diagnostic read
    PT_101_Value : REAL;        // Pressure transmitter value (psi)
    TT_101_Value : REAL;        // Temperature transmitter value (°C)
END_VAR

VAR_OUTPUT
    PRV_101_Active : BOOL;      // Pressure relief valve active
    FV_101_Open : BOOL;         // Feed valve open/close status
    Reboiler_Valve_Open : BOOL; // Reboiler heating valve open/close status
END_VAR

// Constants for thresholds
CONST
    MAX_PRESSURE : REAL := 120.0; // Maximum allowable pressure (psi)
    MIN_PRESSURE : REAL := 50.0;  // Minimum allowable pressure (psi)
    MAX_TEMPERATURE : REAL := 180.0; // Maximum allowable temperature (°C)
END_CONST

// Main execution logic
METHOD Execute : BOOL
BEGIN
    // Initialize outputs
    PRV_101_Active := FALSE;
    FV_101_Open := TRUE;
    Reboiler_Valve_Open := TRUE;

    // Check pressure conditions
    IF PT_101_Value > MAX_PRESSURE THEN
        PRV_101_Active := TRUE; // Activate pressure relief valve
        FV_101_Open := FALSE;   // Close feed valve to prevent further pressurization
    ELSIF PT_101_Value < MIN_PRESSURE THEN
        FV_101_Open := FALSE;   // Close feed valve to prevent underpressure
    END_IF;

    // Check temperature condition
    IF TT_101_Value > MAX_TEMPERATURE THEN
        Reboiler_Valve_Open := FALSE; // Close reboiler heating valve to prevent overheating
    END_IF;

    RETURN TRUE;
END_METHOD

END_PROGRAM
