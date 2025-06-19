FEED --> FV-101 --> C-101 --> BOTTOMS --> E-101 (Reboiler)
                               |
                               |--> VAPOR --> E-102 (Condenser) --> DISTILLATE

Instrumentation:
- PT-101: Pressure Transmitter on C-101 overhead
- TT-101: Temperature Transmitter on E-101 outlet
- LT-101: Level Transmitter on C-101 bottom
- FV-101: Feed Valve (controlled input)
- PRV-101: Pressure Relief Valve (safety actuator on overhead)
- HV-101: Reboiler Heating Valve (noted in logic, assumed actuator)

Control and Interlocks:
- Normal control: FV-101 modulates based on feed demand
- Safety interlocks:
  • If PT-101 > 120 psi → open PRV-101 (pressure relief)
  • If PT-101 < 50 psi → close FV-101 (prevent overfeeding at low pressure)
  • If TT-101 > 180°C → close HV-101 (prevent overheating in reboiler)

  FUNCTION_BLOCK DistillationColumnInterlocks
VAR_INPUT
    PT_101 : REAL;        // Pressure reading in psi
    TT_101 : REAL;        // Temperature reading in Celsius
    Execute : BOOL;       // Trigger for running logic
END_VAR

VAR_OUTPUT
    PRV_101_Open : BOOL;  // Command to open pressure relief valve
    FV_101_Close : BOOL;  // Command to close feed valve
    HV_101_Close : BOOL;  // Command to close reboiler heating valve
END_VAR

VAR
    InterlockEnabled : BOOL;
END_VAR

// Interlock logic execution
IF Execute THEN
    InterlockEnabled := TRUE;

    // Overpressure protection
    IF PT_101 > 120.0 THEN
        PRV_101_Open := TRUE;
    ELSE
        PRV_101_Open := FALSE;
    END_IF;

    // Underpressure feed isolation
    IF PT_101 < 50.0 THEN
        FV_101_Close := TRUE;
    ELSE
        FV_101_Close := FALSE;
    END_IF;

    // Overtemperature protection
    IF TT_101 > 180.0 THEN
        HV_101_Close := TRUE;
    ELSE
        HV_101_Close := FALSE;
    END_IF;

ELSE
    // Reset outputs if not executing
    InterlockEnabled := FALSE;
    PRV_101_Open := FALSE;
    FV_101_Close := FALSE;
    HV_101_Close := FALSE;
END_IF;
