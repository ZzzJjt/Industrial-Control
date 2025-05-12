```
PROGRAM ConveyorBelt
VAR
    speed : INT; // Variable to store the speed of the conveyor belt
END_VAR

METHOD SetSpeed(newSpeed : INT) : BOOL;
    speed := newSpeed;
    RETURN TRUE;
END_METHOD

METHOD GetSpeed() : INT;
    RETURN speed;
END_METHOD

PROGRAM ReversibleConveyorBelt INHERITS ConveyorBelt
METHOD ReverseDirection() : BOOL;
    speed := -speed; // Reverse the speed
    RETURN TRUE;
END_METHOD

FUNCTION_BLOCK ControlConveyor
VAR_INPUT
    conveyor : ANY;
END_VAR
VAR
    speed : INT;
END_VAR

speed := conveyor.GetSpeed(); // Uses polymorphism to call the correct GetSpeed method
conveyor.SetSpeed(speed * 2); // Calls the correct SetSpeed method

```
