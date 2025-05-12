```
FUNCTION_BLOCK Timer
VAR_INPUT
    enable : BOOL; // Enable the timer
    reset : BOOL; // Reset the timer
    duration : TIME; // Duration of the timer
END_VAR
VAR_OUTPUT
    timedOut : BOOL; // Indicates if the timer has timed out
END_VAR
VAR
    elapsed : TIME; // Elapsed time
    running : BOOL; // Indicates if the timer is running
    startTime : TIME; // Start time of the timer
END_VAR

// Initialize variables
elapsed := T#0s;
running := FALSE;

// Handle reset condition
IF reset THEN
    running := FALSE;
    timedOut := FALSE;
ELSIF enable AND NOT running THEN
    // Start the timer
    running := TRUE;
    startTime := CURRENT_TIME;
ELSIF enable AND running THEN
    // Update the elapsed time
    elapsed := TIME_TO_NUM(CURRENT_TIME) - TIME_TO_NUM(startTime);
    IF elapsed >= TIME_TO_NUM(duration) THEN
        // Timer has timed out
        running := FALSE;
        timedOut := TRUE;
    END_IF;
END_IF;

RETURN;

END_FUNCTION_BLOCK

PROGRAM ExampleProgram
VAR
    startSignal : BOOL := FALSE; // Signal to start the timer
    stopSignal : BOOL := FALSE; // Signal to stop/reset the timer
    duration : TIME := T#5s; // Duration of the timer
    timeoutSignal : BOOL; // Output indicating if the timer has timed out
BEGIN
    // Toggle the start signal every 10 seconds (for demonstration purposes)
    startSignal := startSignal XOR (CURRENT_TIME >= T#10s);
    
    // Toggle the stop signal every 20 seconds (for demonstration purposes)
    stopSignal := stopSignal XOR (CURRENT_TIME >= T#20s);
    
    // Call the Timer function block
    Timer(enable:=startSignal, reset:=stopSignal, duration:=duration, timedOut:=timeoutSignal);
    
    // Output the timeout signal status
    WRITE('Timer has timed out: ', timeoutSignal);
END_PROGRAM
```
