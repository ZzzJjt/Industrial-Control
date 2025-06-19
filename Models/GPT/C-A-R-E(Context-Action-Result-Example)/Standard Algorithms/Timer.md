FUNCTION_BLOCK FB_DynamicTimer
VAR_INPUT
    Start      : BOOL;     // Rising-edge triggered start
    Stop       : BOOL;     // Immediate stop
    PresetTime : TIME;     // Duration to time
END_VAR

VAR_OUTPUT
    Done         : BOOL;   // TRUE when timer completes
    Running      : BOOL;   // TRUE while timer is active
    RemainingTime: TIME;   // Time left until done
END_VAR

VAR
    StartEdge    : BOOL;   // Rising edge memory
    Elapsed      : TIME;   // Internal elapsed time tracker
    StartTick    : TIME;   // System time at start
    NowTick      : TIME;   // Current system time
    FirstCycle   : BOOL;   // Latch for first active cycle
END_VAR

// Rising-edge detection
StartEdge := Start AND NOT StartEdge;

// Timer logic
NowTick := TIME(); // Read system time

IF Stop THEN
    // Reset everything if Stop is activated
    Running := FALSE;
    Done := FALSE;
    Elapsed := T#0s;
    RemainingTime := T#0s;
    FirstCycle := TRUE;
ELSIF StartEdge THEN
    // Start new timing
    StartTick := NowTick;
    Running := TRUE;
    Done := FALSE;
    FirstCycle := TRUE;
END_IF;

IF Running THEN
    IF FirstCycle THEN
        StartTick := NowTick;
        FirstCycle := FALSE;
    END_IF;

    Elapsed := NowTick - StartTick;

    IF Elapsed >= PresetTime THEN
        Running := FALSE;
        Done := TRUE;
        RemainingTime := T#0s;
    ELSE
        Done := FALSE;
        RemainingTime := PresetTime - Elapsed;
    END_IF;
END_IF;
