PROGRAM PLC_PRG
TITLE 'Conveyor Feedforward Control – Based on Predicted Load'

(*
    Description:
    A feedforward control system for a conveyor belt that adjusts motor speed
    proactively based on predicted load data from upstream sensors.

    Features:
    - Uses real-time load prediction to adjust conveyor speed before load arrives
    - Prevents material jams or underutilized capacity
    - Supports tunable gain for adaptability
    - Includes safety limits on conveyor speed
    - Modular design suitable for hybrid control strategies

    Safety & Performance:
    - Reduces mechanical stress by avoiding abrupt speed changes
    - Increases throughput by matching speed to expected load
    - Can be combined with PID feedback for optimal performance
*)

// Constants and Tuning Parameters
CONST
    // Base and limit speeds
    BASE_SPEED     : REAL := 1.0;   // Default conveyor speed (m/s)
    MIN_SPEED      : REAL := 0.5;   // Minimum allowable speed (m/s)
    MAX_SPEED      : REAL := 2.0;   // Maximum allowable speed (m/s)

    // Max expected load for scaling purposes
    MAX_EXPECTED_LOAD : REAL := 100.0;  // kg or volume units

    // Feedforward gain (tunable): increase in speed per unit of load
    GAIN_FF        : REAL := 0.02; // m/s per kg
END_CONST

VAR
    // Inputs
    Predicted_Load : REAL := 0.0;     // Upstream sensor data (kg or volume units)

    // Outputs
    Conveyor_Speed : REAL := BASE_SPEED;

    // Internal variables
    Speed_Adjustment : REAL := 0.0;
END_VAR

// === MAIN LOGIC ===

// Calculate the required speed based on predicted load
Speed_Adjustment := BASE_SPEED + GAIN_FF * Predicted_Load;

// Clamp the calculated speed within safe operating range
IF Speed_Adjustment > MAX_SPEED THEN
    Conveyor_Speed := MAX_SPEED;
ELSIF Speed_Adjustment < MIN_SPEED THEN
    Conveyor_Speed := MIN_SPEED;
ELSE
    Conveyor_Speed := Speed_Adjustment;
END_IF;

// Conveyor_Speed is now ready to be sent to the motor controller
// e.g., via analog output or communication protocol like Modbus

END_PROGRAM
