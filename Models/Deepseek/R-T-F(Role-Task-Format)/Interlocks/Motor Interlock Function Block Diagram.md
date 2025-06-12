FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL; // Running status of Equipment 1 (TRUE = Running, FALSE = Stopped)
    Equipment2Running : BOOL; // Running status of Equipment 2 (TRUE = Running, FALSE = Stopped)
    Equipment3Running : BOOL; // Running status of Equipment 3 (TRUE = Running, FALSE = Stopped)
END_VAR

VAR_OUTPUT
    AllowStart : BOOL; // Output to allow motor start (TRUE = Allowed, FALSE = Blocked)
END_VAR

METHOD Execute : BOOL
BEGIN
    // Determine if all associated equipment is stopped
    AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;

    RETURN TRUE;
END_METHOD

END_FUNCTION_BLOCK

PROGRAM MainControl
VAR
    MotorLock : MotorInterlock;
    StartButtonPressed : BOOL; // Input from start button
    MotorRunning : BOOL;       // Output indicating motor status
END_VAR

// Simulated input values for demonstration purposes
MotorLock.Equipment1Running := FALSE; // Assume Equipment 1 is stopped
MotorLock.Equipment2Running := FALSE; // Assume Equipment 2 is stopped
MotorLock.Equipment3Running := FALSE; // Assume Equipment 3 is stopped

// Call the MotorInterlock function block
MotorLock.Execute();

// Check if the motor can start based on the interlock logic
IF StartButtonPressed AND MotorLock.AllowStart THEN
    MotorRunning := TRUE; // Start the motor
ELSE
    MotorRunning := FALSE; // Do not start the motor
END_IF;
