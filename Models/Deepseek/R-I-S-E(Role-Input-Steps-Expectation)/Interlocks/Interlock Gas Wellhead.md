FUNCTION_BLOCK FB_SubseaWellheadInterlock
VAR_INPUT
    // Sensor Inputs
    rPT_101: REAL;         // Pressure Transmitter - psi
    rTT_101: REAL;         // Temperature Transmitter - °C
    rFT_101: REAL;         // Flow Meter - m³/hr

    bResetButton: BOOL;    // Manual Reset Button (NO contact)
    bSystemEnabled: BOOL;  // System Running State
END_VAR

VAR_OUTPUT
    // Actuator Outputs
    bMV_101_Open: BOOL;     // Master Valve MV-101 – TRUE = Open, FALSE = Closed
    bShutdownActive: BOOL;  // Indicates that system is in SHUTDOWN state
    bAlarmTriggered: BOOL;  // General alarm signal to SCADA or HMI
    bIsolateValves: BOOL;   // Command to close all isolation valves
    bLockoutSystem: BOOL;   // Prevents restart until manual reset
END_VAR

VAR
    // Internal Flags
    bPressureHighTrip: BOOL := FALSE;
    bFlowLowTrip: BOOL := FALSE;
    bTempHighTrip: BOOL := FALSE;

    bAnyTripCondition: BOOL := FALSE;
    bShutdownLatched: BOOL := FALSE;
END_VAR

// Initialize outputs at start of scan
bMV_101_Open := FALSE;
bIsolateValves := FALSE;
bLockoutSystem := FALSE;
bAlarmTriggered := FALSE;
bShutdownActive := FALSE;

// Evaluate trip conditions
bPressureHighTrip := rPT_101 > 1500.0;           // Overpressure condition
bFlowLowTrip := rFT_101 < 20.0;                  // Low flow indicates possible leak
bTempHighTrip := rTT_101 > 120.0;                // High temp condition

// Detect any active trip condition
bAnyTripCondition := bPressureHighTrip OR bFlowLowTrip OR bTempHighTrip;

// Latch the shutdown condition if any fault occurs
IF bAnyTripCondition THEN
    bShutdownLatched := TRUE;
END_IF;

// Manual Emergency Stop can also set the latch
IF NOT bSystemEnabled THEN
    bShutdownLatched := TRUE;
END_IF;

// Only update outputs if latched in SHUTDOWN state
IF bShutdownLatched THEN
    bShutdownActive := TRUE;
    bMV_101_Open := FALSE;      // Close master valve
    bIsolateValves := TRUE;     // Isolate all process lines
    bLockoutSystem := TRUE;     // Prevent auto-restart
    bAlarmTriggered := TRUE;    // Activate alarm
END_IF;

// Manual reset required to clear shutdown latch
IF bResetButton THEN
    bShutdownLatched := FALSE;
    bLockoutSystem := FALSE;
    bIsolateValves := FALSE;
    bAlarmTriggered := FALSE;
    bShutdownActive := FALSE;
END_IF;

PROGRAM PLC_PRG
VAR
    WellheadInterlock: FB_SubseaWellheadInterlock;
    PT_101_Value: REAL := AIN1;       // Example analog input mapping
    TT_101_Value: REAL := AIN2;
    FT_101_Value: REAL := AIN3;
    Reset_Button: BOOL := DI1;
    System_Running: BOOL := NOT ESD_ACTIVE;
END_VAR

// Call function block with real-world inputs
WellheadInterlock(
    rPT_101 := PT_101_Value,
    rTT_101 := TT_101_Value,
    rFT_101 := FT_101_Value,
    bResetButton := Reset_Button,
    bSystemEnabled := System_Running
);
