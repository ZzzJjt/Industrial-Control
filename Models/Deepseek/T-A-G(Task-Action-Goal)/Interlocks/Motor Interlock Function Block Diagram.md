PROGRAM MAIN_PROGRAM
VAR
    MI : MotorInterlock; // Instance of MotorInterlock function block
    StartCommand : BOOL; // Command to start the motor
    MotorRunning : BOOL; // Status of the motor
END_VAR

VAR_INPUT
    Equipment1Status : BOOL; // Input from sensor indicating Equipment 1 status
    Equipment2Status : BOOL; // Input from sensor indicating Equipment 2 status
    Equipment3Status : BOOL; // Input from sensor indicating Equipment 3 status
END_VAR

// Update the function block inputs
MI.Equipment1Running := Equipment1Status;
MI.Equipment2Running := Equipment2Status;
MI.Equipment3Running := Equipment3Status;

// Execute the function block
MI.Execute();

// Use the AllowStart output to control the motor
IF StartCommand AND MI.AllowStart THEN
    MotorRunning := TRUE;
ELSE
    MotorRunning := FALSE;
END_IF;



