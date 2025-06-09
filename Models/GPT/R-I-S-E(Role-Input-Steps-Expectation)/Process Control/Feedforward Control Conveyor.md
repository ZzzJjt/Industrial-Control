FUNCTION_BLOCK ConveyorFeedforwardControl
VAR_INPUT
    Predicted_Load  : REAL;             // Upstream sensor prediction (kg or volume)
END_VAR

VAR_OUTPUT
    Conveyor_Speed  : REAL;             // Speed command to motor controller
END_VAR

VAR
    Base_Speed      : REAL := 1.0;      // Base conveyor speed in m/s
    Gain_FF         : REAL := 0.02;     // Gain factor per unit of predicted load
    Max_Speed       : REAL := 2.0;      // Max allowed speed
    Min_Speed       : REAL := 0.5;      // Min allowed speed
END_VAR

// --- Feedforward Speed Calculation ---
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

// --- Safety Clamping ---
IF Conveyor_Speed > Max_Speed THEN
    Conveyor_Speed := Max_Speed;
ELSIF Conveyor_Speed < Min_Speed THEN
    Conveyor_Speed := Min_Speed;
END_IF;
