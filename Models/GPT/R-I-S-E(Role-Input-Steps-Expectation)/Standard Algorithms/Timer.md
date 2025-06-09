FUNCTION_BLOCK FB_FlexibleTimer
VAR_INPUT
    Start      : BOOL;
    Stop       : BOOL;
    PresetTime : TIME;
END_VAR

VAR_OUTPUT
    Running       : BOOL := FALSE;
    Done          : BOOL := FALSE;
    RemainingTime : TIME := T#0s;
END_VAR

VAR
    StartTime     : TIME := T#0s;
    ElapsedTime   : TIME := T#0s;
    SystemNow     : TIME;
    StartEdge     : BOOL := FALSE;
    PrevStart     : BOOL := FALSE;
END_VAR

// Rising edge detection for Start
StartEdge := Start AND NOT PrevStart;
PrevStart := Start;

// Get current system time (assuming a system call like TIME() is available)
SystemNow := TIME(); // Replace with PLC-specific system time retrieval

// Handle Stop input — reset timer
IF Stop THEN
    Running := FALSE;
    Done := FALSE;
    StartTime := T#0s;
    ElapsedTime := T#0s;
    RemainingTime := T#0s;

// On Start rising edge — capture start time
ELSIF StartEdge THEN
    StartTime := SystemNow;
    Running := TRUE;
    Done := FALSE;

// If timer is running — update elapsed and check for Done
ELSIF Running THEN
    ElapsedTime := SystemNow - StartTime;

    IF ElapsedTime >= PresetTime THEN
        Done := TRUE;
        Running := FALSE;
        RemainingTime := T#0s;
    ELSE
        RemainingTime := PresetTime - ElapsedTime;
    END_IF
END_IF
