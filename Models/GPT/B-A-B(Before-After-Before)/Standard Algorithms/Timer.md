FUNCTION_BLOCK FB_FlexibleTimer
VAR_INPUT
    Start      : BOOL;     // Starts the timer
    Stop       : BOOL;     // Stops and resets the timer
    PresetTime : TIME;     // Target duration to time
END_VAR

VAR_OUTPUT
    Running       : BOOL;  // TRUE when timer is active
    Done          : BOOL;  // TRUE when PresetTime has elapsed
    RemainingTime : TIME;  // Optional: Time left before Done
END_VAR

VAR
    StartTime   : TIME;    // Timestamp when timer started
    ElapsedTime : TIME;    // Accumulated time
    CurrentTime : TIME;    // For delta calculation
    LastTime    : TIME;    // Tracks last system time
    Active      : BOOL;    // Internal flag
END_VAR

// System time update (replace with actual system time call in your PLC)
CurrentTime := TIME(); // Pseudo-call – use your platform’s current time

IF Stop THEN
    Active := FALSE;
    Running := FALSE;
    Done := FALSE;
    ElapsedTime := T#0s;
    RemainingTime := PresetTime;
ELSIF Start THEN
    IF NOT Active THEN
        Active := TRUE;
        StartTime := CurrentTime;
        ElapsedTime := T#0s;
    ELSE
        // Accumulate elapsed time using time difference
        ElapsedTime := ElapsedTime + (CurrentTime - LastTime);
    END_IF
END_IF

LastTime := CurrentTime;

// Handle outputs
IF Active THEN
    Running := TRUE;
    IF ElapsedTime >= PresetTime THEN
        Done := TRUE;
        RemainingTime := T#0s;
    ELSE
        Done := FALSE;
        RemainingTime := PresetTime - ElapsedTime;
    END_IF
ELSE
    Running := FALSE;
    Done := FALSE;
    RemainingTime := PresetTime;
END_IF
