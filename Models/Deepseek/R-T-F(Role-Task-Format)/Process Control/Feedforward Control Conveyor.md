PROGRAM ConveyorFeedforwardControl
VAR
    // Inputs
    Predicted_Load : REAL; // kg or volume units from sensor
    Base_Speed : REAL := 1.0; // Minimum conveyor speed (m/s)
    Gain_FF : REAL := 0.02; // Speed increase per unit load

    // Output
    Conveyor_Speed : REAL;
END_VAR

// Feedforward control logic
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

// Clamp speed within operating limits
IF Conveyor_Speed > 2.0 THEN
    Conveyor_Speed := 2.0; // Maximum conveyor speed (m/s)
ELSIF Conveyor_Speed < 0.5 THEN
    Conveyor_Speed := 0.5; // Minimum conveyor speed (m/s)
END_IF;

// Conveyor_Speed is sent to motor control interface
// Example: SetMotorSpeed(Conveyor_Speed); // Placeholder function to set motor speed
