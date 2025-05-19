FUNCTION_BLOCK PIDControllerFB
VAR_INPUT
    Enable: BOOL;               // Enable PID operation
    Reset: BOOL;                // Reset internal states
    Setpoint: REAL;             // Desired process value
    ProcessVariable: REAL;      // Measured process value
    Kp: REAL := 1.0;            // Proportional gain
    Ki: REAL := 0.1;            // Integral gain
    Kd: REAL := 0.01;           // Derivative gain
    DeltaT: REAL := 0.1;        // Time step (s)
    MinOutput: REAL := -100.0;  // Minimum output limit
    MaxOutput: REAL := 100.0;   // Maximum output limit
    TuningMode: BOOL;           // Enable runtime gain tuning
END_VAR

VAR_OUTPUT
    Output: REAL;               // Control output
    Error: REAL;                // Current error (Setpoint - ProcessVariable)
    Integral: REAL;             // Integral term for diagnostics
    Derivative: REAL;           // Derivative term for diagnostics
    ValidInput: BOOL;           // TRUE if inputs are valid
END_VAR

VAR
    PrevProcessVariable: REAL := 0.0; // Previous process value for derivative
    Windup: BOOL案を共有する。Windup := FALSE;           // Anti-windup flag
    Initialized: BOOL := FALSE; // Tracks initialization state
    TempOutput: REAL;          // Temporary output before clamping
END_VAR

// Reset outputs and state when disabled
IF NOT Enable THEN
    Output := 0.0;
    Error := 0.0;
    Integral := 0.0;
    Derivative := 0.0;
    PrevProcessVariable := 0.0;
    Windup := FALSE;
    Initialized := FALSE;
    ValidInput := FALSE;
    RETURN;
END_IF;

// Input validation
ValidInput := TRUE;
IF DeltaT <= 0.0 OR MinOutput >= MaxOutput OR Kp < 0.0 OR Ki < 0.0 OR Kd < 0.0 THEN
    ValidInput := FALSE;
    Output := 0.0;
    Error := 0.0;
    Integral := 0.0;
    Derivative := 0.0;
    RETURN;
END_IF;

// Handle reset
IF Reset THEN
    Integral := 0.0;
    PrevProcessVariable := ProcessVariable;
    Windup := FALSE;
    Initialized := TRUE;
END_IF;

// PID control logic
IF ValidInput AND (Initialized OR Reset) THEN
    // Calculate error
    Error := Setpoint - ProcessVariable;

    // Proportional term
    TempOutput := Kp * Error;

    // Integral term with anti-windup
    IF NOT Windup THEN
        Integral := Integral + (Ki * Error * DeltaT);
    END_IF;
    TempOutput := TempOutput + Integral;

    // Derivative term (based on process variable to reduce setpoint change impact)
    IF Initialized THEN
        Derivative := (PrevProcessVariable - ProcessVariable) / DeltaT;
    ELSE
        Derivative := 0.0;
    END_IF;
    TempOutput := TempOutput - Kd * Derivative;

    // Output clamping with anti-windup
    IF TempOutput > MaxOutput THEN
        Output := MaxOutput;
        Windup := TRUE;
    ELSIF TempOutput < MinOutput THEN
        Output := MinOutput;
        Windup := TRUE;
    ELSE
        Output := TempOutput;
        Windup := FALSE;
    END_IF;

    // Update previous process variable
    PrevProcessVariable := ProcessVariable;
    Initialized := TRUE;
ELSE
    // Initialize on first valid cycle
    IF ValidInput AND NOT Initialized THEN
        PrevProcessVariable := ProcessVariable;
        Integral := 0.0;
        Derivative := 0.0;
        Error := Setpoint - ProcessVariable;
        Output := Kp * Error; // Start with proportional only
        Initialized := TRUE;
        ValidInput := TRUE;
    END_IF;
END_IF;

// Optional tuning mode: Allow runtime gain adjustment
IF TuningMode AND ValidInput THEN
    // Gains can be modified externally during operation
    // No additional logic needed; ensure Kp, Ki, Kd are updated safely
END_IF;
END_FUNCTION_BLOCK
