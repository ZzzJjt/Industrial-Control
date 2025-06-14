VAR
    // Input from load prediction sensor
    Predicted_Load   : REAL;        // Predicted material load [kg or volume units]

    // Feedforward control parameters
    Base_Speed       : REAL := 1.0; // Minimum base conveyor speed [m/s]
    Gain_FF          : REAL := 0.02; // Gain factor [m/s per unit load]

    // Output to conveyor motor controller
    Conveyor_Speed   : REAL;        // Final calculated conveyor speed [m/s]

    // Speed limits
    Max_Speed        : REAL := 2.0; // Maximum conveyor speed [m/s]
    Min_Speed        : REAL := 0.5; // Minimum conveyor speed [m/s]
END_VAR

// -------------------------
// Feedforward Control Logic
// -------------------------

// Step 1: Calculate conveyor speed based on predicted load
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

// Step 2: Clamp speed within safe operational bounds
IF Conveyor_Speed > Max_Speed THEN
    Conveyor_Speed := Max_Speed;
ELSIF Conveyor_Speed < Min_Speed THEN
    Conveyor_Speed := Min_Speed;
END_IF;

// Conveyor_Speed is now ready to be sent to the motor controller
