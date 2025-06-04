PROGRAM ConveyorFeedforwardControl
VAR
    // --- Input Variables ---
    Predicted_Load   : REAL;       // Predicted load from upstream sensor (e.g., kg)
    Base_Speed       : REAL := 1.0; // Minimum conveyor speed (m/s)
    Max_Load         : REAL := 100.0; // Reference maximum load (optional for normalization)
    Gain_FF          : REAL := 0.02;  // Feedforward gain (m/s per kg)

    // --- Output Variable ---
    Conveyor_Speed   : REAL;       // Actual conveyor speed output
END_VAR

// --- Feedforward Control Logic ---
// Calculate target conveyor speed
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

// --- Clamping Logic ---
// Ensure speed stays within safe operational limits
IF Conveyor_Speed > 2.0 THEN
    Conveyor_Speed := 2.0;         // Maximum speed limit
ELSIF Conveyor_Speed < 0.5 THEN
    Conveyor_Speed := 0.5;         // Minimum speed limit
END_IF

// Conveyor_Speed is used to drive the conveyor motor
