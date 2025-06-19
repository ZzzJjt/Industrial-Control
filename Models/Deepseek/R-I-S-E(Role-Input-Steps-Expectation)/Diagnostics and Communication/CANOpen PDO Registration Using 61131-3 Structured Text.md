FUNCTION_BLOCK CAN_REGISTER_COBID
VAR_INPUT
    COBID: DWORD := 0;        // CANOpen message ID
    REGISTER: BOOL := FALSE;  // TRUE = Register, FALSE = Deregister
    ENABLE: BOOL := TRUE;     // Optional enable input
END_VAR

VAR_OUTPUT
    STATUS: BYTE := 0;         // Status code
    ERROR: BOOL := FALSE;      // General error flag
    ERROR_CODE: BYTE := 0;     // Specific error code
END_VAR

VAR
    prev_register: BOOL := FALSE;
    prev_cobid: DWORD := 0;
END_VAR

METHOD PRIVATE ClearAllRegistrations: BOOL
ClearAllRegistrations := CAN_ClearAllRegistrations();
END_METHOD

METHOD PRIVATE RegisterSingle: BOOL
IF NOT CAN_Register(COBID) THEN
    ERROR_CODE := 1; // Registration failed
    RETURN FALSE;
END_IF;
RETURN TRUE;
END_METHOD

METHOD PRIVATE DeregisterSingle: BOOL
IF NOT CAN_Deregister(COBID) THEN
    ERROR_CODE := 2; // Deregistration failed
    RETURN FALSE;
END_IF;
RETURN TRUE;
END_METHOD

// Reset status
ERROR := FALSE;
ERROR_CODE := 0;

// Only execute if enabled
IF NOT ENABLE THEN
    STATUS := 0; // Idle
    RETURN;
END_IF;

// Prevent duplicate execution on same edge
IF (prev_register = REGISTER) AND (prev_cobid = COBID) THEN
    RETURN;
END_IF;

// Handle special case: deregister all
IF NOT REGISTER AND COBID = 0 THEN
    IF NOT ClearAllRegistrations() THEN
        ERROR := TRUE;
        STATUS := 3; // All deregistration failed
    ELSE
        STATUS := 4; // All deregistered successfully
    END_IF;

// Handle normal registration/deregistration
ELSIF REGISTER THEN
    IF NOT RegisterSingle() THEN
        ERROR := TRUE;
        STATUS := 1; // Registration failed
    ELSE
        STATUS := 2; // Registered successfully
    END_IF;

ELSE
    IF NOT DeregisterSingle() THEN
        ERROR := TRUE;
        STATUS := 5; // Deregistration failed
    ELSE
        STATUS := 6; // Deregistered successfully
    END_IF;
END_IF;

// Save current state
prev_register := REGISTER;
prev_cobid := COBID;

// STATUS CODES
// 0 - Idle / No action
// 1 - Registration failed
// 2 - Registered successfully
// 3 - Clear all registrations failed
// 4 - Cleared all registrations successfully
// 5 - Deregistration failed
// 6 - Deregistered successfully

// ERROR_CODES
// 0 - No error
// 1 - Failed to register COBID
// 2 - Failed to deregister COBID
// 3 - Invalid COBID (if validation added)
// 4 - Duplicate registration attempt
