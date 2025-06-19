FEED --> [FV-101] --> [C-101: Distillation Column] --> Overhead Vapor --> [E-102: Condenser] --> Product Out
                                            |
                                            |
                                         [E-101: Reboiler]
                                            |
                                         Bottoms Product Out

Instrumentation:
- PT-101: Measures pressure at top of C-101
- TT-101: Measures temperature at base near E-101
- LT-101: Measures liquid level inside C-101
- FV-101: Feed valve to control flow into column
- PRV-101: Pressure relief valve at column top

Control Relationships:
- PT-101 influences PRV-101 (open if overpressure)
- PT-101 influences FV-101 (close if underpressure)
- TT-101 influences reboiler heater (shut off if overtemperature)

FUNCTION_BLOCK FB_DistillationInterlocks
VAR_INPUT
    Pressure : REAL;      // From PT-101 (psi)
    Temperature : REAL;   // From TT-101 (°C)
    Execute : BOOL;       // Rising-edge trigger
END_VAR

VAR_OUTPUT
    PRV_101_Open : BOOL;         // TRUE opens pressure relief valve
    FV_101_Close : BOOL;         // TRUE closes feed valve
    Reboiler_Heater_Off : BOOL;  // TRUE turns off reboiler heater
    ErrorID : DWORD;             // Error or fault identifier
END_VAR

VAR
    RisingEdgeExecute : R_TRIG;
END_VAR

// Rising edge detection to avoid repeated action on static values
RisingEdgeExecute(CLK := Execute);

// Default outputs
PRV_101_Open := FALSE;
FV_101_Close := FALSE;
Reboiler_Heater_Off := FALSE;
ErrorID := 0;

// Interlock Logic
IF RisingEdgeExecute.Q THEN
    // Interlock 1: Overpressure
    IF Pressure > 120.0 THEN
        PRV_101_Open := TRUE;
    END_IF;

    // Interlock 2: Underpressure
    IF Pressure < 50.0 THEN
        FV_101_Close := TRUE;
    END_IF;

    // Interlock 3: Overtemperature
    IF Temperature > 180.0 THEN
        Reboiler_Heater_Off := TRUE;
    END_IF;

    // Optional error handling (example)
    IF (Pressure < 0.0) OR (Temperature < 0.0) THEN
        ErrorID := 16#00000001; // Invalid sensor reading
    END_IF;
END_IF;
