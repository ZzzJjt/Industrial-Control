VAR
    // Inputs
    StartCommand : BOOL;
    SafetyOK : BOOL;
    Temp_OK : BOOL; // TRUE when temperature reaches 85°C–90°C
    AllMotorsReady : BOOL;

    // Outputs
    PressRollsOn : BOOL := FALSE;
    ConveyorsOn : BOOL := FALSE;
    NipPressure : REAL := 0.0; // [kN/m]
    PressSpeed : REAL := 0.0;  // [m/min]

    // Internal
    StartupState : INT := 0; // State machine
    PressureRampTimer : TON;
    SpeedRampTimer : TON;
    RampTime : TIME := T#60s; // 60 seconds ramp time
    TimeStep : REAL := 0.1;   // 100 ms cycle time
    TargetPressure : REAL := 250.0; // [kN/m]
    TargetSpeed : REAL := 500.0;    // [m/min]
END_VAR

// --- STARTUP SEQUENCE STATE MACHINE ---
CASE StartupState OF
    0: // Idle - waiting for start and safety ready
        IF StartCommand AND SafetyOK THEN
            StartupState := 1;
        END_IF

    1: // Start Press Rolls
        PressRollsOn := TRUE;
        IF AllMotorsReady THEN
            StartupState := 2;
            PressureRampTimer(IN := TRUE, PT := RampTime);
        END_IF

    2: // Ramp-up Nip Pressure
        IF PressureRampTimer.Q THEN
            NipPressure := TargetPressure;
            PressureRampTimer(IN := FALSE);
            StartupState := 3;
            SpeedRampTimer(IN := TRUE, PT := RampTime);
        ELSE
            NipPressure := NipPressure + (TargetPressure * TimeStep / REAL_TO_TIME(RampTime));
        END_IF

    3: // Ramp-up Speed
        IF SpeedRampTimer.Q AND Temp_OK THEN
            PressSpeed := TargetSpeed;
            ConveyorsOn := TRUE;
            SpeedRampTimer(IN := FALSE);
            StartupState := 4;
        ELSE
            PressSpeed := PressSpeed + (TargetSpeed * TimeStep / REAL_TO_TIME(RampTime));
        END_IF

    4: // Normal Operation
        // Maintain outputs, monitor system
        // Add further logic as needed
END_CASE
