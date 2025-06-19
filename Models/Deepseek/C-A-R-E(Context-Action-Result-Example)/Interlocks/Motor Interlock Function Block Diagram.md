FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL; // Status indicating if Equipment 1 is running
    Equipment2Running : BOOL; // Status indicating if Equipment 2 is running
    Equipment3Running : BOOL; // Status indicating if Equipment 3 is running
END_VAR

VAR_OUTPUT
    AllowStart : BOOL; // Output indicating if the motor can be started
END_VAR

// Main execution logic
AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;



