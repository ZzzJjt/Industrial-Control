FUNCTION_BLOCK FlexibleTimerFB
VAR_INPUT
    Enable: BOOL;               // Enable timer operation
    Start: BOOL;                // Start timer (rising edge)
    Stop: BOOL;                 // Stop timer
    PresetTime: TIME := T#1s;   // Configurable timer duration
END_VAR

VAR_OUTPUT
    Running: BOOL;              // TRUE when timer is counting
    Done: BOOL;                 // TRUE when timer completes
    RemainingTime: TIME;        // Time remaining until completion
    Error: BOOL;                // TRUE if invalid input (e.g., negative PresetTime)
END_VAR

VAR
    StartEdge: BOOL;            // Previous state of Start for edge detection
    StartTime: TIME;            // Time when timer started
    ElapsedTime: TIME;          // Time elapsed since start
    ValidInput: BOOL;           // Input validation flag
    TimerActive: BOOL;          // Internal state for timer operation
END_VAR

// Reset outputs when disabled
IF NOT Enable THEN
    Running := FALSE;
    Done := FALSE;
    RemainingTime := T#0s;
    Error := FALSE;
    TimerActive := FALSE;
    StartEdge := FALSE;
    ElapsedTime := T#0s;
    RETURN;
END_IF;

// Input validation
ValidInput := TRUE;
IF PresetTime <= T#0s THEN
    ValidInput := FALSE;
    Error := TRUE;
    Running := FALSE;
    Done := FALSE;
    RemainingTime := T#0s;
    TimerActive := FALSE;
    RETURN;
END_IF;

// Detect rising edge of Start
IF Start AND NOT StartEdge AND NOT Stop AND ValidInput THEN
    TimerActive := TRUE;
    StartTime := TIME(); // Capture current system time
    ElapsedTime := T#0s;
    Running := TRUE;
    Done := FALSE;
END_IF;
StartEdge := Start; // Update edge detection state

// Timer logic
IF ValidInput AND TimerActive THEN
    IF Stop THEN
        // Stop timer
        TimerActive := FALSE;
        Running := FALSE;
        Done := FALSE;
        ElapsedTime := T#0s;
        RemainingTime := PresetTime;
    ELSE
        // Update elapsed time
        ElapsedTime := TIME() - StartTime;

        // Check for completion
        IF ElapsedTime >= PresetTime THEN
            Running := FALSE;
            Done := TRUE;
            TimerActive := FALSE;
            RemainingTime := T#0s;
        ELSE
            Running := TRUE;
            Done := FALSE;
            // Calculate remaining time
            IF PresetTime > ElapsedTime THEN
                RemainingTime := PresetTime - ElapsedTime;
            ELSE
                RemainingTime := T#0s;
            END_IF;
        END_IF;
    END_IF;
ELSE
    // Ensure outputs are reset if not active
    Running := FALSE;
    Done := FALSE;
    RemainingTime := PresetTime;
END_IF;
END_FUNCTION_BLOCK
