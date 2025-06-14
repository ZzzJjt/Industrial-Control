FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL;
    Equipment2Running : BOOL;
    Equipment3Running : BOOL;
    StartRequest : BOOL;         // External request to start the motor
END_VAR

VAR_OUTPUT
    AllowStart : BOOL;           // Interlock output: TRUE = safe to start
    MotorStart : BOOL;           // Final motor start signal after interlock
END_VAR

VAR
    AllEquipmentStopped : BOOL;
END_VAR

// Interlock condition: All equipment must be OFF
AllEquipmentStopped := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;

// Output interlock signal
AllowStart := AllEquipmentStopped;

// Allow motor start only if interlock is satisfied
MotorStart := AllowStart AND StartRequest;
