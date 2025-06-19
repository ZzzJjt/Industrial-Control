FUNCTION_BLOCK FB_CustomTimer
VAR_INPUT
    Start       : BOOL;     // Start trigger
    Stop        : BOOL;     // Reset or stop
    PresetTime  : TIME;     // Target timing duration
END_VAR

VAR_OUTPUT
    Running      : BOOL := FALSE;   // TRUE while timing
    Done         : BOOL := FALSE;   // TRUE when time has elapsed
    RemainingTime: TIME := T#0s;    // Optional: remaining time
END_VAR

VAR
    StartEdge       : BOOL := FALSE; // Rising edge detection
    StartEdgeOld    : BOOL := FALSE;
    StartTime       : TIME := T#0s;
    ElapsedTime     : TIME := T#0s;
END_VAR

// ============ Main Logic ============

// Rising edge detection for Start
StartEdge := Start AND NOT StartEdgeOld;
StartEdgeOld := Start;

// Stop resets everything
IF Stop THEN
    Running := FALSE;
    Done := FALSE;
    StartTime := T#0s;
    ElapsedTime := T#0s;
    RemainingTime := PresetTime;

ELSIF StartEdge THEN
    // Capture current time at rising edge
    StartTime := TIME();  // System time in TIME
    Running := TRUE;
    Done := FALSE;

ELSIF Running THEN
    // Calculate elapsed time
    ElapsedTime := TIME() - StartTime;

    // Check if timer has reached preset
    IF ElapsedTime >= PresetTime THEN
        Running := FALSE;
        Done := TRUE;
        RemainingTime := T#0s;
    ELSE
        Done := FALSE;
        RemainingTime := PresetTime - ElapsedTime;
    END_IF
END_IF
