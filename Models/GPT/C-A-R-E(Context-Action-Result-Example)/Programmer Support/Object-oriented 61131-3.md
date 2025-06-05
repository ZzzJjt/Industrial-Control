// Base actuator
FUNCTION_BLOCK FB_Actuator
METHOD Start : BOOL
    Start := TRUE;
END_METHOD
END_FUNCTION_BLOCK

// Valve extends actuator
FUNCTION_BLOCK FB_Valve EXTENDS FB_Actuator
METHOD Start : BOOL
    // Override with valve-specific logic
    Start := TRUE; // Add valve control logic
END_METHOD
END_FUNCTION_BLOCK

INTERFACE IStartable
METHOD Start : BOOL;
END_INTERFACE

FUNCTION_BLOCK FB_Motor IMPLEMENTS IStartable
METHOD Start : BOOL
    Start := TRUE;
END_METHOD
END_FUNCTION_BLOCK
