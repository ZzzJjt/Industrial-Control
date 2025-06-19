PROGRAM PLC_PRG
VAR
    // System Control Flags
    bMachineReady         : BOOL := FALSE;
    bStartSequence        : BOOL := FALSE;
    bShutdownRequested    : BOOL := FALSE;

    // Heating Stations
    aHeaterActive         : ARRAY [1..8] OF BOOL := [FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE];
    aHeaterAtTemp         : ARRAY [1..8] OF BOOL := [FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE];

    // Cooling Stations
    aCoolerActive         : ARRAY [1..8] OF BOOL := [FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE];

    // Cutters
    bHorizontalCutterOn   : BOOL := FALSE;
    bVerticalCutterOn     : BOOL := FALSE;

    // Feeder Units
    rTensionA             : REAL := 0.0;  // Measured tension
    rTensionB             : REAL := 0.0;
    bFeederAActive        : BOOL := FALSE;
    bFeederBActive        : BOOL := FALSE;

    // Timers
    tmrStartup            : TON;
    tmrCooldown           : TON;
    tmrTensionRelease     : TON;

    // State Machine
    eState                : E_MACHINE_STATE := STATE_IDLE;
    eNextState            : E_MACHINE_STATE := STATE_IDLE;

    // Parameters
    rMinTension           : REAL := 5.0;  // Minimum acceptable tension (N)
    rMaxTension           : REAL := 15.0; // Maximum safe tension (N)
END_VAR

CASE eState OF
    STATE_IDLE:
        IF bStartSequence THEN
            eNextState := STATE_STARTUP_SEQUENCE;
            tmrStartup(IN := TRUE, PT := T#5s); // Delay before starting heaters
        END_IF;

    STATE_STARTUP_SEQUENCE:
        TurnOnAllHeaters();
        eNextState := STATE_WAIT_FOR_TEMP;

    STATE_WAIT_FOR_TEMP:
        IF AllHeatersAtTemp() THEN
            eNextState := STATE_FEEDER_START;
        END_IF;

    STATE_FEEDER_START:
        StartFeedersWithTensionControl();
        eNextState := STATE_CUTTER_SYNC;

    STATE_CUTTER_SYNC:
        SyncCuttersWithMaterial();
        eNextState := STATE_RUNNING;

    STATE_RUNNING:
        bMachineReady := TRUE;

    STATE_SHUTDOWN_SEQUENCE:
        StopCutters();
        eNextState := STATE_COOL_DOWN;

    STATE_COOL_DOWN:
        TurnOffAllCoolersAfterDelay(T#30s);
        eNextState := STATE_TENSION_RELEASE;

    STATE_TENSION_RELEASE:
        ReleaseTensionGradually(T#10s);
        eNextState := STATE_FINAL_STOP;

    STATE_FINAL_STOP:
        StopAllSystems();
        eNextState := STATE_IDLE;
END_CASE;

// Advance state if timer or condition is met
IF tmrStartup.Q THEN
    tmrStartup(IN := FALSE);
    eState := eNextState;
END_IF;

METHOD StartFeedersWithTensionControl
bFeederAActive := TRUE;
bFeederBActive := TRUE;

// Ensure tension stays within limits
IF rTensionA < rMinTension OR rTensionB < rMinTension THEN
    RaiseLowTensionAlarm();
ELSIF rTensionA > rMaxTension OR rTensionB > rMaxTension THEN
    RaiseHighTensionAlarm();
END_IF;
