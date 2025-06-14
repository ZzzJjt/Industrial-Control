FUNCTION_BLOCK TimerFunctionBlock
VAR_INPUT
    Start : BOOL;            // Triggers the timer
    Stop : BOOL;             // Resets or halts the timer
    PresetTime : TIME;       // Sets the duration to time
END_VAR

VAR_OUTPUT
    Running : BOOL;          // Indicates the timer is active
    Done : BOOL;             // Becomes TRUE when the preset time is reached
    RemainingTime : TIME;    // Time left before timeout
END_VAR

VAR
    StartTime : TIME;         // Time when the timer started
    ElapsedTime : TIME;       // Elapsed time since the timer started
    LastStartTime : TIME;     // Last recorded start time for edge detection
    LastStartEdge : BOOL;     // Previous state of the Start input for edge detection
    LastStopEdge : BOOL;      // Previous state of the Stop input for edge detection
END_VAR

METHOD Execute;
BEGIN
    // Initialize outputs
    Running := FALSE;
    Done := FALSE;
    RemainingTime := T#0s;

    // Detect rising edge of Start input
    IF Start AND NOT LastStartEdge THEN
        // Start the timer
        StartTime := TONF(T#0s);
        LastStartTime := StartTime;
        Running := TRUE;
        Done := FALSE;
    END_IF;

    // Detect rising edge of Stop input
    IF Stop AND NOT LastStopEdge THEN
        // Reset the timer
        Running := FALSE;
        Done := FALSE;
    END_IF;

    // Update the last states for edge detection
    LastStartEdge := Start;
    LastStopEdge := Stop;

    // If the timer is running, calculate elapsed time
    IF Running THEN
        ElapsedTime := TONF(T#0s) - StartTime;

        // Check if the preset time has been reached
        IF ElapsedTime >= PresetTime THEN
            Running := FALSE;
            Done := TRUE;
        ELSE
            RemainingTime := PresetTime - ElapsedTime;
        END_IF;
    END_IF;
END_METHOD



