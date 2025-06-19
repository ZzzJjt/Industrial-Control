FUNCTION_BLOCK FB_SubseaESD
VAR_INPUT
    PT_Pressure       : REAL;   // Pressure (psi)
    FT_FlowRate       : REAL;   // Flow rate (e.g., SCFM)
    TT_Temperature    : REAL;   // Temperature (°C)
    RESET             : BOOL;   // Manual reset input (e.g., from HMI)
END_VAR

VAR_OUTPUT
    MV_101            : BOOL;   // Master Valve Open/Closed (TRUE = Open)
    SHUTDOWN          : BOOL;   // System shutdown status flag
    Alarm_HighPressure: BOOL;
    Alarm_LowFlow     : BOOL;
    Alarm_HighTemp    : BOOL;
END_VAR

VAR
    Shutdown_Latch    : BOOL := FALSE;
    SAFE_FLOW_MIN     : REAL := 100.0;     // Minimum safe flow threshold
    PRESSURE_MAX      : REAL := 1500.0;    // Maximum safe pressure
    TEMPERATURE_MAX   : REAL := 120.0;     // Maximum safe temperature
END_VAR

// Reset logic (manual reset only)
IF RESET AND NOT (PT_Pressure > PRESSURE_MAX OR FT_FlowRate < SAFE_FLOW_MIN OR TT_Temperature > TEMPERATURE_MAX) THEN
    Shutdown_Latch := FALSE;
END_IF;

// Interlock triggers
IF PT_Pressure > PRESSURE_MAX THEN
    Shutdown_Latch := TRUE;
    Alarm_HighPressure := TRUE;
END_IF;

IF FT_FlowRate < SAFE_FLOW_MIN THEN
    Shutdown_Latch := TRUE;
    Alarm_LowFlow := TRUE;
END_IF;

IF TT_Temperature > TEMPERATURE_MAX THEN
    Shutdown_Latch := TRUE;
    Alarm_HighTemp := TRUE;
END_IF;

// Apply shutdown and safe state logic
IF Shutdown_Latch THEN
    MV_101 := FALSE;     // Close master valve
    SHUTDOWN := TRUE;    // Set global shutdown flag
ELSE
    MV_101 := TRUE;      // Open master valve (normal operation)
    SHUTDOWN := FALSE;
    Alarm_HighPressure := FALSE;
    Alarm_LowFlow := FALSE;
    Alarm_HighTemp := FALSE;
END_IF;
