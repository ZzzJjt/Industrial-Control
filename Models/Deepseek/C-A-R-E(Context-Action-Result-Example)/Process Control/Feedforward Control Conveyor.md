PROGRAM FeedforwardControlConveyor
VAR_INPUT
    Predicted_Load : REAL; // Load from upstream sensor (e.g., kg)
END_VAR

VAR_OUTPUT
    Conveyor_Speed : REAL; // Output speed to drive the conveyor motor (m/s)
END_VAR

VAR
    Base_Speed : REAL := 1.0; // Minimum conveyor speed (m/s)
    Max_Load : REAL := 100.0; // Load scaling reference (kg)
    Gain_FF : REAL := 0.02;   // Feedforward gain (speed per kg)
    Desired_Speed : REAL;
END_VAR

// Feedforward speed calculation
Desired_Speed := Base_Speed + Gain_FF * Predicted_Load;

// Clamp speed to operational limits
IF Desired_Speed > 2.0 THEN
    Conveyor_Speed := 2.0;
ELSIF Desired_Speed < 0.5 THEN
    Conveyor_Speed := 0.5;
ELSE
    Conveyor_Speed := Desired_Speed;
END_IF;

// Inline comments explaining the logic:
// The program implements feedforward control for regulating conveyor belt speed.
// It receives real-time input from a load prediction sensor and calculates the desired speed using a feedforward gain.
// The calculated speed is then clamped to ensure it remains within safe operational limits.
// This approach enables preemptive speed adjustment, improving efficiency and reducing mechanical wear.



