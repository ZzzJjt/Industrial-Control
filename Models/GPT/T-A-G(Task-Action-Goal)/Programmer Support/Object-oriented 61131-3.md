FUNCTION_BLOCK FB_Pump
VAR
    Speed : REAL;
    Status : BOOL;
END_VAR

METHOD Start : BOOL
Start := TRUE;
Status := TRUE;

✅ Advantages of OOP in Industrial Automation

Feature
Benefit
Modularity
Isolate logic in reusable components
Abstraction
Hide implementation details behind interfaces
Encapsulation
Bundle data and methods together for clarity
Reusability
Reuse common logic in multiple applications (e.g., pumps, valves)
Maintainability
Easier updates and debugging

FUNCTION_BLOCK FB_Valve
VAR
    IsOpen : BOOL;
END_VAR

METHOD Open
IsOpen := TRUE;

FUNCTION_BLOCK FB_SafetyValve EXTENDS FB_Valve
VAR
    SafetyLock : BOOL;
END_VAR

METHOD Open
IF NOT SafetyLock THEN
    SUPER^.Open(); // Call base method
END_IF

INTERFACE IDevice
METHOD Start : BOOL;

FUNCTION_BLOCK FB_Motor IMPLEMENTS IDevice
METHOD Start : BOOL
Start := TRUE;

FUNCTION_BLOCK FB_Fan IMPLEMENTS IDevice
METHOD Start : BOOL
Start := TRUE;

PROGRAM Main
VAR
    device : IDevice;  // Base reference
    motor  : FB_Motor;
    fan    : FB_Fan;
END_VAR

device := motor;
device.Start();  // Calls FB_Motor.Start

device := fan;
device.Start();  // Calls FB_Fan.Start
