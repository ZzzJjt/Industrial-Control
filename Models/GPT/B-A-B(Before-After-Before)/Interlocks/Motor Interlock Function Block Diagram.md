FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL;
    Equipment2Running : BOOL;
    Equipment3Running : BOOL;
END_VAR

VAR_OUTPUT
    AllowStart : BOOL;
END_VAR

// Interlock Logic: Allow motor start only if all equipment are stopped
AllowStart := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;


PROGRAM PLC_Main
VAR
    MyMotorInterlock : MotorInterlock;

    // Equipment status signals
    EQ1 : BOOL := FALSE;
    EQ2 : BOOL := FALSE;
    EQ3 : BOOL := FALSE;

    // Output to control motor start permission
    MotorCanStart : BOOL;
END_VAR

// Connect inputs
MyMotorInterlock.Equipment1Running := EQ1;
MyMotorInterlock.Equipment2Running := EQ2;
MyMotorInterlock.Equipment3Running := EQ3;

// Use output
MotorCanStart := MyMotorInterlock.AllowStart;
