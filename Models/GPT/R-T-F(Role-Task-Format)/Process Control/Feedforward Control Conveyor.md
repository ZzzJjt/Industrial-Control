PROGRAM FeedforwardConveyorControl
VAR
    // --- Inputs ---
    Predicted_Load  : REAL;            // Predicted load from sensor (e.g., kg or volume units)
    Base_Speed      : REAL := 1.0;     // Base conveyor speed in m/s
    Gain_FF         : REAL := 0.02;    // Feedforward gain (speed per unit of load)

    // --- Output ---
    Conveyor_Speed  : REAL;            // Final conveyor speed command
END_VAR

// --- Feedforward Control Calculation ---
// Adjust conveyor speed based on predicted material load
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

// --- Safety Clamping Logic ---
// Clamp conveyor speed within operational bounds: 0.5 m/s to 2.0 m/s
IF Conveyor_Speed > 2.0 THEN
    Conveyor_Speed := 2.0;
ELSIF Conveyor_Speed < 0.5 THEN
    Conveyor_Speed := 0.5;
END_IF

// Conveyor_Speed is now ready to be applied to the motor control interface
