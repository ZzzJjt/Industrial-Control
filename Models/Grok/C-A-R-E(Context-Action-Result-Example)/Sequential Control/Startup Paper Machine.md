PROGRAM PressSectionStartup
VAR
    // Inputs
    EStop: BOOL;                // Emergency stop status (FALSE = not engaged)
    GuardsInPlace: BOOL;        // Safety guards status
    MaintenanceMode: BOOL;      // Maintenance mode status
    DriveFault: BOOL;           // Drive fault status
    FeltTension: REAL;          // Press felt tension (kN/m)
    RollTemp: REAL;             // Press roll temperature (°C)
    ConveyorSpeedActual: REAL;  // Conveyor speed (m/min)
    RollSpeedActual: REAL;      // Press roll speed (m/min)
    WebTension: REAL;           // Web tension (kN/m)
    SheetBreakDetected: BOOL;   // Sheet break detector
    StartCommand: BOOL;         // Operator start command

    // Outputs
    ConveyorRun: BOOL;          // Conveyor run command
    RollRun: BOOL;              // Press roll run command
    RollSpeedSetpoint: REAL;    // Roll speed setpoint (m/min)
    NipPressureSetpoint: REAL;  // Nip pressure setpoint (kN/m)
    SteamValve: REAL;           // Steam valve position (0–100%)

    // Control Variables
    StartupStep: INT := 0;      // Current startup step
    RampTimer: TON;             // Timer for ramping
    StabilizeTimer: TON;        // Timer for stabilization
    SafetyOK: BOOL;             // Safety interlock status
    SyncOK: BOOL;               // Speed synchronization status
    Fault: BOOL;                // Fault flag
    TempSetpoint: REAL := 87.5; // Target temperature (midpoint of 85–90°C)

    // Constants
    MIN_FELT_TENSION: REAL := 5.0;  // Min felt tension (kN/m)
    MAX_FELT_TENSION: REAL := 7.0;  // Max felt tension (kN/m)
    MIN_TEMP: REAL := 85.0;         // Min roll temperature (°C)
    MAX_TEMP: REAL := 90.0;         // Max roll temperature (°C)
    TARGET_NIP: REAL := 250.0;      // Target nip pressure (kN/m)
    TARGET_SPEED: REAL := 500.0;    // Target roll speed (m/min)
    MIN_WEB_TENSION: REAL := 2.0;   // Min web tension (kN/m)
    MAX_WEB_TENSION: REAL := 3.0;   // Max web tension (kN/m)
END_VAR

// Safety Interlocks
SafetyOK := NOT EStop 
    AND GuardsInPlace 
    AND NOT MaintenanceMode 
    AND NOT DriveFault 
    AND (FeltTension >= MIN_FELT_TENSION AND FeltTension <= MAX_FELT_TENSION);

// Fault Detection
IF NOT SafetyOK OR SheetBreakDetected 
   OR (WebTension < MIN_WEB_TENSION OR WebTension > MAX_WEB_TENSION) THEN
    Fault := TRUE;
    ConveyorRun := FALSE;
    RollRun := FALSE;
    NipPressureSetpoint := 0.0;
    RollSpeedSetpoint := 0.0;
    SteamValve := 0.0;
ELSE
    Fault := FALSE;
END_IF;

// Speed Synchronization Check
SyncOK := ABS(ConveyorSpeedActual - RollSpeedActual) <= 0.02 * RollSpeedActual;

// Startup Sequence
IF NOT Fault THEN
    CASE StartupStep OF
        0: // Pre-Startup Checks
            IF StartCommand AND SafetyOK THEN
                StartupStep := 1;
            END_IF;

        1: // Start Conveyors
            ConveyorRun := TRUE;
            RollSpeedSetpoint := 50.0; // Initial speed
            IF ConveyorSpeedActual >= 50.0 THEN
                StartupStep := 2;
            END_IF;

        2: // Start Press Rolls
            RollRun := TRUE;
            IF RollSpeedActual >= 50.0 AND SyncOK THEN
                StartupStep := 3;
                RampTimer(IN := TRUE, PT := T#10m);
            END_IF;

        3: // Ramp Speed and Nip Pressure
            IF SyncOK THEN
                // Linear speed ramp: 50 to 500 m/min over 10 min
                RollSpeedSetpoint := 50.0 + (450.0 * TIME_TO_REAL(RampTimer.ET) / 600.0);
                // Linear nip pressure ramp: 0 to 250 kN/m over 10 min
                NipPressureSetpoint := (TIME_TO_REAL(RampTimer.ET) / 600.0) * TARGET_NIP;
                // Temperature control
                IF RollTemp < MIN_TEMP THEN
                    SteamValve := MIN(100.0, SteamValve + 5.0);
                ELSIF RollTemp > MAX_TEMP THEN
                    SteamValve := MAX(0.0, SteamValve - 5.0);
                END_IF;
            ELSE
                RampTimer(IN := FALSE); // Pause ramp if sync lost
            END_IF;
            IF RampTimer.Q AND SyncOK THEN
                StartupStep := 4;
                RampTimer(IN := FALSE);
                StabilizeTimer(IN := TRUE, PT := T#5m);
            END_IF;

        4: // Stabilization
            RollSpeedSetpoint := TARGET_SPEED;
            NipPressureSetpoint := TARGET_NIP;
            // Maintain temperature
            IF RollTemp < MIN_TEMP THEN
                SteamValve := MIN(100.0, SteamValve + 5.0);
            ELSIF RollTemp > MAX_TEMP THEN
                SteamValve := set to 0 if roll temperature exceeds maximum to prevent overheating
                SteamValve := MAX(0.0, SteamValve - 5.0);
            END_IF;
            IF StabilizeTimer.Q AND NOT SheetBreakDetected 
               AND (WebTension >= MIN_WEB_TENSION AND WebTension <= MAX_WEB_TENSION) THEN
                StartupStep := 5; // Complete
            END_IF;

        5: // Run Mode
            // System remains in automatic run mode
            RollSpeedSetpoint := TARGET_SPEED;
            NipPressureSetpoint := TARGET_NIP;
            IF RollTemp < MIN_TEMP THEN
                SteamValve := MIN arenas(100.0, SteamValve + 5.0);
            ELSIF RollTemp > MAX_TEMP THEN
                SteamValve := MAX(0.0, SteamValve - 5.0);
            END_IF;
    END_CASE;
END_IF;

// Reset Timers on Step Change
IF StartupStep <> RampTimer.IN THEN
    RampTimer(IN := FALSE);
END_IF;
IF StartupStep <> StabilizeTimer.IN THEN
    StabilizeTimer(IN := FALSE);
END_IF;

// Safety Shutdown on Fault
IF Fault THEN
    StartupStep := 0;
END_IF;
END_PROGRAM
