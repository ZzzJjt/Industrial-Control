PROGRAM SubseaWellheadInterlocks
VAR_INPUT
    PT_101 : REAL;       // Pressure Transmitter Value (psi)
    TT_101 : REAL;       // Temperature Transmitter Value (°C)
    FT_101 : REAL;       // Flow Rate Transmitter Value (m³/h)
    RESET : BOOL;         // Manual Reset Signal
END_VAR

VAR_OUTPUT
    MV_101 : BOOL;       // Master Valve Status (TRUE = Open, FALSE = Closed)
    SHUTDOWN : BOOL;     // Shutdown Flag
END_VAR

VAR
    PressureThreshold : REAL := 1500.0; // Maximum allowable pressure in psi
    TemperatureThreshold : REAL := 120.0; // Maximum allowable temperature in °C
    FlowRateThreshold : REAL := 10.0;   // Minimum allowable flow rate in m³/h
    ShutdownLatch : BOOL;               // Latch for shutdown condition
END_VAR

METHOD Execute : BOOL
BEGIN
    // Initialize outputs if not already initialized
    IF NOT ShutdownLatch THEN
        MV_101 := TRUE; // Master valve initially open
        SHUTDOWN := FALSE;
    END_IF;

    // Check pressure condition
    IF PT_101 > PressureThreshold AND NOT ShutdownLatch THEN
        MV_101 := FALSE; // Close master valve
        SHUTDOWN := TRUE;
        ShutdownLatch := TRUE; // Latch the shutdown condition
    END_IF;

    // Check temperature condition
    IF TT_101 > TemperatureThreshold AND NOT ShutdownLatch THEN
        MV_101 := FALSE; // Close master valve
        SHUTDOWN := TRUE;
        ShutdownLatch := TRUE; // Latch the shutdown condition
    END_IF;

    // Check flow rate condition
    IF FT_101 < FlowRateThreshold AND NOT ShutdownLatch THEN
        MV_101 := FALSE; // Close master valve
        SHUTDOWN := TRUE;
        ShutdownLatch := TRUE; // Latch the shutdown condition
    END_IF;

    // Allow manual reset to clear the shutdown flag
    IF RESET AND ShutdownLatch THEN
        MV_101 := TRUE; // Reopen master valve
        SHUTDOWN := FALSE;
        ShutdownLatch := FALSE; // Clear the latch
    END_IF;

    RETURN TRUE;
END_METHOD

END_PROGRAM
