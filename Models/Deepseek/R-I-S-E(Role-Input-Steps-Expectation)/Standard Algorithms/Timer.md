FUNCTION_BLOCK FlexibleTimerFB
VAR_INPUT
    PresetTime : TIME;     // Configurable preset time
    Start : BOOL;          // Start signal
    Stop : BOOL;           // Stop signal
END_VAR

VAR_OUTPUT
    Running : BOOL;        // Timer is active
    Done : BOOL;           // Timer has reached its preset time
    RemainingTime : TIME;  // Remaining time until preset time is reached
END_VAR

VAR
    StartTime : TIME;       // Time when the timer started
    ElapsedTime : TIME;     // Elapsed time since the timer started
    LastStart : BOOL := FALSE; // Previous state of Start input for edge detection
    LastStop : BOOL := FALSE;  // Previous state of Stop input for edge detection
END_VAR

METHOD Execute : VOID
BEGIN
    // Detect rising edge of Start
    IF Start AND NOT LastStart THEN
        StartTime := CURRENT_TIME;
        Running := TRUE;
        Done := FALSE;
    END_IF;

    // Detect rising edge of Stop
    IF Stop AND NOT LastStop THEN
        Running := FALSE;
        Done := FALSE;
        RemainingTime := PresetTime;
    END_IF;

    // Store current state of Start and Stop inputs for next scan cycle
    LastStart := Start;
    LastStop := Stop;

    // If the timer is running, calculate elapsed time
    IF Running THEN
        ElapsedTime := CURRENT_TIME - StartTime;

        // Check if the elapsed time has reached or exceeded the preset time
        IF ElapsedTime >= PresetTime THEN
            Done := TRUE;
            Running := FALSE;
            RemainingTime := T#0S;
        ELSE
            Done := FALSE;
            RemainingTime := PresetTime - ElapsedTime;
        END_IF;
    ELSE
        RemainingTime := PresetTime;
    END_IF;
END_METHOD

END_FUNCTION_BLOCK

PROGRAM MainProgram
VAR
    FTB : FlexibleTimerFB;
    PresetTime : TIME := T#5S; // Preset time of 5 seconds
    Start : BOOL := FALSE;     // Start signal
    Stop : BOOL := FALSE;      // Stop signal
    Running : BOOL;
    Done : BOOL;
    RemainingTime : TIME;
END_VAR

// Simulate user inputs
IF CurrentTime MOD 10000 = 0 THEN // Every 10 seconds
    Start := TRUE;
    Stop := FALSE;
ELSIF CurrentTime MOD 15000 = 0 THEN // Every 15 seconds
    Start := FALSE;
    Stop := TRUE;
END_IF;

// Configure the function block with sample inputs
FTB(PresetTime := PresetTime,
    Start := Start,
    Stop := Stop);

// Call the function block
FTB.Execute();

// Use the outputs
Running := FTB.Running;
Done := FTB.Done;
RemainingTime := FTB.RemainingTime;

// Example usage: Print or log the results
// Note: In a real PLC environment, you would typically use outputs to control other parts of the system
IF Done THEN
    // Timer has completed
ELSEIF Running THEN
    // Timer is running
ELSE
    // Timer is stopped
END_IF;
END_PROGRAM
