FUNCTION_BLOCK FB_CustomTimer
VAR_INPUT
    Start      : BOOL;   // Start trigger
    Stop       : BOOL;   // Stop/reset trigger
    PresetTime : TIME;   // Configurable duration
END_VAR

VAR_OUTPUT
    Running       : BOOL := FALSE; // TRUE while timing
    Done          : BOOL := FALSE; // TRUE when PresetTime elapsed
    RemainingTime : TIME := T#0s;  // Optional: time left until done
END_VAR

VAR
    StartTime     : TIME := T#0s;  // Timestamp when timer started
    ElapsedTime   : TIME := T#0s;  // Time since Start
    CurrentTime   : TIME := T#0s;  // Current system time
    StartEdge     : BOOL := FALSE; // Internal rising edge detection
    StartPrev     : BOOL := FALSE; // Previous state of Start
END_VAR

// === Edge Detection for Start ===
StartEdge := Start AND NOT StartPrev;
StartPrev := Start;

// === Stop Logic (has priority) ===
IF Stop THEN
    Running := FALSE;
    Done := FALSE;
    RemainingTime := PresetTime;
    StartTime := T#0s;
    ElapsedTime := T#0s;

ELSIF StartEdge THEN
    // Rising edge of Start
    StartTime := TIME();       // Capture current system time
    Running := TRUE;
    Done := FALSE;

ELSIF Running THEN
    // Timer is active: compute elapsed and check timeout
    CurrentTime := TIME();
    ElapsedTime := CurrentTime - StartTime;

    IF ElapsedTime >= PresetTime THEN
        Done := TRUE;
        Running := FALSE;
        RemainingTime := T#0s;
    ELSE
        RemainingTime := PresetTime - ElapsedTime;
    END_IF
END_IF
