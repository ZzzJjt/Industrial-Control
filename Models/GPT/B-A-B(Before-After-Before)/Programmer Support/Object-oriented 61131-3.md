// Actuator – Abstract base class
{FB}
FUNCTION_BLOCK FB_Actuator
VAR
    isActive : BOOL := FALSE;
END_VAR

METHOD PUBLIC Start : BOOL
    isActive := TRUE;
    Start := isActive;

METHOD PUBLIC Stop : BOOL
    isActive := FALSE;
    Stop := isActive;

METHOD PUBLIC ABSTRACT Status : STRING;

// ValveController – Inherits from FB_Actuator
{FB}
FUNCTION_BLOCK FB_ValveController EXTENDS FB_Actuator
VAR
    valvePosition : INT := 0; // 0 = closed, 100 = fully open
END_VAR

METHOD PUBLIC SetPosition
VAR_INPUT
    position : INT;
END_VAR
    valvePosition := position;

METHOD PUBLIC OVERRIDE Status : STRING
VAR_OUTPUT
    Status : STRING;
END_VAR
    IF isActive THEN
        Status := CONCAT('Valve active, position: ', INT_TO_STRING(valvePosition));
    ELSE
        Status := 'Valve is inactive';
    END_IF

  INTERFACE IControl
    METHOD Start : BOOL;
    METHOD Stop : BOOL;
    METHOD Status : STRING;

VAR
device : IControl; // Interface reference
valve : FB_ValveController;
END_VAR

device := valve; // Polymorphic assignment
device.Start();  // Will call FB_ValveController’s Start()
device.Status(); // Will call overridden method
