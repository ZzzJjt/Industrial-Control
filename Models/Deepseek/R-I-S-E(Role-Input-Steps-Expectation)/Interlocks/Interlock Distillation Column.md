Equipment:
- C-101: Distillation Column
- E-101: Reboiler
- E-102: Condenser

Sensors:
- PT-101: Pressure Transmitter on C-101
- TT-101: Temperature Transmitter on C-101
- LT-101: Level Transmitter on Reflux Drum (connected to top of C-101)

Actuators:
- FV-101: Feed Valve controlling feed into C-101
- PRV-101: Pressure Relief Valve on C-101

Control Loops:
- PT-101 monitors pressure in C-101; controls PRV-101 for overpressure protection.
- TT-101 monitors temperature in C-101; used to control heat input to E-101.
- LT-101 monitors level in reflux drum; used for level control and safety interlocks.
- FV-101 regulates feed flow into C-101 based on process requirements and safety interlocks.

FUNCTION_BLOCK FB_DistillationInterlock
VAR_INPUT
    // Input Signals (Sensor Values)
    rPressure: REAL;   // From PT-101
    rTemperature: REAL; // From TT-101
    rLevel: REAL;      // From LT-101
END_VAR

VAR_OUTPUT
    // Output Actions (Actuator Commands)
    bPRV_Open: BOOL;       // Command PRV-101 to open
    bFV_Close: BOOL;       // Command FV-101 to close
    bReboiler_Shutdown: BOOL; // Shutdown command for E-101
END_VAR

// Internal Variables
VAR
    bAnyCriticalFault: BOOL;
END_VAR

// Initialization of outputs
bPRV_Open := FALSE;
bFV_Close := FALSE;
bReboiler_Shutdown := FALSE;

// Detect if any critical condition exists
bAnyCriticalFault := 
    rPressure > 120 OR rPressure < 50 OR rTemperature > 180;

// General emergency response if any fault detected
IF bAnyCriticalFault THEN
    bFV_Close := TRUE;
    bReboiler_Shutdown := TRUE;
END_IF;

// Overpressure Protection
IF rPressure > 120 THEN
    bPRV_Open := TRUE;
END_IF;

// Low Pressure Response
IF rPressure < 50 THEN
    bFV_Close := TRUE;
END_IF;

// High Temperature Protection
IF rTemperature > 180 THEN
    bReboiler_Shutdown := TRUE;
END_IF;
