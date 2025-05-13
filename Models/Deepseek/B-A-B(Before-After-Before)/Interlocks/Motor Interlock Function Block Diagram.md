[Equipment1Running] -->| NOT |\
                        AND   => [AllowStart := TRUE]
[Equipment2Running] -->| NOT |/
                        AND
[Equipment3Running] -->| NOT |


FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running: BOOL;  // Status of equipment 1
    Equipment2Running: BOOL;  // Status of equipment 2
    Equipment3Running: BOOL;  // Status of equipment 3
END_VAR

VAR_OUTPUT
    AllowStart: BOOL;        // TRUE if it's safe to start the motor
END_VAR

// Interlock logic
AllowStart :=
    NOT Equipment1Running AND
    NOT Equipment2Running AND
    NOT Equipment3Running;
END_FUNCTION_BLOCK

PROGRAM PLC_PRG
VAR
    MotorInterlock_1: MotorInterlock;
    MotorStartButton: BOOL := FALSE;
    MotorRunning: BOOL := FALSE;
END_VAR

// Call function block
MotorInterlock_1(
    Equipment1Running := PumpRunning,
    Equipment2Running := ConveyorMoving,
    Equipment3Running := AgitatorActive
);

// Only allow motor start if interlock permits
IF MotorStartButton AND MotorInterlock_1.AllowStart THEN
    MotorRunning := TRUE;
END_IF;

// Additional logic for stopping the motor...

