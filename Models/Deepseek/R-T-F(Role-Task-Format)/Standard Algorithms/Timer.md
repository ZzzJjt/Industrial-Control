PROGRAM MainProgram
VAR
    TimerFB : FB_CustomizableTimer;
    Start : BOOL := FALSE; // Start signal for the timer
    Stop : BOOL := FALSE; // Stop signal for the timer
    PresetTime : TIME := T#5s; // Preset time for the timer (e.g., 5 seconds)
    Running : BOOL;
    Done : BOOL;
    RemainingTime : TIME;
END_VAR

// Simulate user input to start the timer
IF TP() > T#2s THEN
    Start := TRUE;
END_IF;

// Simulate user input to stop the timer after some time
IF TP() > T#8s THEN
    Stop := TRUE;
END_IF;

// Set the inputs for the timer
TimerFB.Start := Start;
TimerFB.Stop := Stop;
TimerFB.PresetTime := PresetTime;

// Call the timer function block
TimerFB();

// Get the results
Running := TimerFB.Running;
Done := TimerFB.Done;
RemainingTime := TimerFB.RemainingTime;

// Output the results
// For demonstration purposes, assume there's a way to display or log the results
// Display(Running); // Hypothetical display function
// Display(Done); // Hypothetical display function
// Display(TO_STRING(RemainingTime)); // Hypothetical display function

FUNCTION_BLOCK FB_CustomizableTimer
VAR_INPUT
    Start : BOOL; // Input to start the timer
    Stop : BOOL; // Input to stop the timer
    PresetTime : TIME; // Configurable preset time for the timer
END_VAR

VAR_OUTPUT
    Running : BOOL := FALSE; // Output indicating if the timer is running
    Done : BOOL := FALSE; // Output indicating if the timer has reached the preset time
    RemainingTime : TIME; // Optional output indicating remaining time
END_VAR

VAR
    StartTime : TIME; // Time when the timer was started
    ElapsedTime : TIME; // Elapsed time since the timer started
    TriggerEdge : BOOL := FALSE; // Internal flag for edge detection on Start input
    LastStart : BOOL := FALSE; // Previous state of Start input for edge detection
END_VAR

// Method to detect rising edge on Start input
METHOD DetectStartEdge(this : REFERENCE TO FB_CustomizableTimer) : BOOL
BEGIN
    VAR
        EdgeDetected : BOOL;
    END_VAR

    EdgeDetected := this.Start AND NOT this.LastStart;
    this.LastStart := this.Start;

    RETURN EdgeDetected;
END_METHOD

// Main method to execute the timer logic
METHOD Execute(this : REFERENCE TO FB_CustomizableTimer) : BOOL
BEGIN
    VAR
        CurrentTime : TIME;
    END_VAR

    // Reset flags
    this.Running := FALSE;
    this.Done := FALSE;

    // Detect rising edge on Start input
    IF this.DetectStartEdge() THEN
        this.TriggerEdge := TRUE;
        this.StartTime := TP(); // Record the start time
    END_IF;

    // Handle Stop input
    IF this.Stop THEN
        this.TriggerEdge := FALSE;
        this.Running := FALSE;
        this.Done := FALSE;
        this.RemainingTime := T#0s;
        RETURN TRUE;
    END_IF;

    // If the timer is triggered and not stopped
    IF this.TriggerEdge THEN
        CurrentTime := TP();
        ElapsedTime := CurrentTime - this.StartTime;

        // Check if the elapsed time has reached the preset time
        IF ElapsedTime >= this.PresetTime THEN
            this.Running := FALSE;
            this.Done := TRUE;
            this.RemainingTime := T#0s;
        ELSE
            this.Running := TRUE;
            this.Done := FALSE;
            this.RemainingTime := this.PresetTime - ElapsedTime;
        END_IF;
    END_IF;

    RETURN TRUE;
END_METHOD

// Main execution method
METHOD MAIN(this : REFERENCE TO FB_CustomizableTimer) : BOOL
BEGIN
    RETURN this.Execute();
END_METHOD
