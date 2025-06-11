PROGRAM PLC_PRG
VAR
    // Startup Flags
    bStartPressStartup      : BOOL := FALSE;
    bStartupComplete        : BOOL := FALSE;
    bEmergencyStop          : BOOL := FALSE;

    // Process Parameters
    rTargetSpeed            : REAL := 500.0;     // m/min
    rTargetNipPressure      : REAL := 250.0;     // kN/m
    rMinTemp                : REAL := 85.0;      // °C
    rMaxTemp                : REAL := 90.0;      // °C

    // Equipment Status
    bSystemReady            : BOOL := FALSE;
    bConveyorRunning        : BOOL := FALSE;
    bRollPreRotating        : BOOL := FALSE;
    bHeaterActive           : BOOL := FALSE;
    bNipEngaged             : BOOL := FALSE;
    bSpeedRamping           : BOOL := FALSE;

    // Sensors
    rCurrentSpeed           : REAL := 0.0;       // Current line speed (m/min)
    rCurrentNipPressure     : REAL := 0.0;       // Current nip pressure (kN/m)
    rCurrentTemp            : REAL := 0.0;       // Current press temperature (°C)

    // Timers
    tmrPreRotationDelay     : TON;
    tmrHeatUp               : TON;
    tmrNipRamp              : TON;
    tmrSpeedRamp            : TON;

    // State Machine
    eState                  : E_PRESS_STARTUP_STEP := STEP_IDLE;

    // Outputs
    rCalculatedSpeedOutput  : REAL := 0.0;       // Output to drives (%)
    rCalculatedNipOutput    : REAL := 0.0;       // Output to hydraulic system (%)
END_VAR

// Emergency Stop Override
IF bEmergencyStop THEN
    eState := STEP_ABORTED;
END_IF;

CASE eState OF
    STEP_IDLE:
        IF bStartPressStartup THEN
            IF CheckSystemReadiness() THEN
                StartConveyorPreRotation();
                eState := STEP_CONVEYOR_PRE_ROTATION;
            END_IF;
        END_IF;

    STEP_CONVEYOR_PRE_ROTATION:
        IF IsPreRotationDone() THEN
            StartPreheat(rMinTemp, rMaxTemp);
            eState := STEP_TEMPERATURE_PREHEAT;
        END_IF;

    STEP_TEMPERATURE_PREHEAT:
        IF IsTemperatureInRange() THEN
            StartNipRamp(rTargetNipPressure);
            eState := STEP_NIP_PRESSURE_RAMP_UP;
        END_IF;

    STEP_NIP_PRESSURE_RAMP_UP:
        IF IsNipAtTarget() THEN
            StartSpeedRamp(rTargetSpeed);
            eState := STEP_SPEED_RAMP_UP;
        END_IF;

    STEP_SPEED_RAMP_UP:
        IF IsSpeedAtTarget() THEN
            bStartupComplete := TRUE;
            eState := STEP_COMPLETE;
        END_IF;

    STEP_COMPLETE:
        ; // No action required — handled by higher-level system

    STEP_ABORTED:
        AbortAllProcesses();
        bStartupComplete := FALSE;
END_CASE;

FUNCTION RampNipPressureOverTime : REAL
VAR_INPUT
    fInitialPressure    : REAL := 0.0;   // Starting pressure
    fFinalPressure      : REAL := 250.0; // Target pressure (kN/m)
    tElapsedTime        : TIME;         // Elapsed time since start
    tTotalDuration      : TIME := T#10m; // Total ramp-up time
END_VAR

IF tElapsedTime >= tTotalDuration THEN
    RampNipPressureOverTime := fFinalPressure;
ELSE
    RampNipPressureOverTime := fInitialPressure + ((fFinalPressure - fInitialPressure) * 
                        (TIME_TO_REAL(tElapsedTime) / TIME_TO_REAL(tTotalDuration)));
END_IF;
