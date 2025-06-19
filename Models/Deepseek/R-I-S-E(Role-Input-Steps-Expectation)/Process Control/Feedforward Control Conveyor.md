FUNCTION_BLOCK FB_ConveyorFeedforwardControl
VAR_INPUT
    // Predicted load from upstream sensor (kg or volume units)
    Predicted_Load : REAL;

    // Base conveyor speed (m/s)
    Base_Speed : REAL := 1.0;

    // Feedforward gain (speed adjustment per unit load)
    Gain_FF : REAL := 0.02;
END_VAR

VAR_OUTPUT
    // Calculated conveyor speed to send to motor controller (m/s)
    Conveyor_Speed : REAL;
END_VAR

VAR
    // Internal logic variables
    Temp_Speed : REAL;
END_VAR

// --- STEP 1: Calculate Conveyor Speed Using Feedforward Control ---
Temp_Speed := Base_Speed + Gain_FF * Predicted_Load;

// --- STEP 2: Limit Speed to Safe Operational Bounds ---
IF Temp_Speed > 2.0 THEN
    Conveyor_Speed := 2.0; // Max speed limit
ELSIF Temp_Speed < 0.5 THEN
    Conveyor_Speed := 0.5; // Min speed limit
ELSE
    Conveyor_Speed := Temp_Speed;
END_IF;

// The Conveyor_Speed variable now contains the adjusted speed value that can be sent to the motor controller.

PROGRAM PLC_PRG
VAR
    ConveyorCtrl : FB_ConveyorFeedforwardControl;

    // Simulated Inputs
    UpstreamLoadPrediction : REAL := 50.0; // Example predicted load value

    // Output
    MotorControllerCommand : REAL;
END_VAR

// Call the function block
ConveyorCtrl(
    Predicted_Load := UpstreamLoadPrediction,
    Base_Speed := 1.0,
    Gain_FF := 0.02,

    Conveyor_Speed => MotorControllerCommand
);

// Now, MotorControllerCommand holds the calculated conveyor speed to be sent to the motor controller.

PROGRAM PLC_PRG
VAR
    ConveyorCtrl : FB_ConveyorFeedforwardControl;

    // Simulated Inputs
    UpstreamLoadPrediction : REAL := 50.0; // Example predicted load value

    // Output
    MotorControllerCommand : REAL;
END_VAR

// Call the function block
ConveyorCtrl(
    Predicted_Load := UpstreamLoadPrediction,
    Base_Speed := 1.0,
    Gain_FF := 0.02,

    Conveyor_Speed => MotorControllerCommand
);

// Now, MotorControllerCommand holds the calculated conveyor speed to be sent to the motor controller.
