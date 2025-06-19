VAR
    // === Inputs ===
    Predicted_Load : REAL;            // Load prediction input (kg or volume)
    Base_Speed : REAL := 1.0;         // Minimum conveyor speed (m/s)
    Max_Load : REAL := 100.0;         // Maximum expected load (kg)

    // === Feedforward Coefficient ===
    Gain_FF : REAL := 0.02;           // Speed increase per kg of load

    // === Output ===
    Conveyor_Speed : REAL;            // Final conveyor speed (m/s)
END_VAR

// === Feedforward Control Calculation ===
Conveyor_Speed := Base_Speed + Gain_FF * Predicted_Load;

// === Clamping Logic to Safe Speed Range ===
IF Conveyor_Speed > 2.0 THEN
    Conveyor_Speed := 2.0;            // Upper limit for safety
ELSIF Conveyor_Speed < 0.5 THEN
    Conveyor_Speed := 0.5;            // Lower limit for reliable operation
END_IF

// === Conveyor_Speed is the control signal to motor drive ===
