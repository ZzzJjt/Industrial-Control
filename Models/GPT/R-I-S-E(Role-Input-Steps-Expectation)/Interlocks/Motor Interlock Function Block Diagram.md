FUNCTION_BLOCK MotorInterlock
VAR_INPUT
    Equipment1Running : BOOL;
    Equipment2Running : BOOL;
    Equipment3Running : BOOL;
END_VAR
VAR_OUTPUT
    AllowStart : BOOL;
END_VAR
VAR
    AllStopped : BOOL;
END_VAR

// Interlock logic: only allow start if ALL equipment is stopped
AllStopped := NOT Equipment1Running AND NOT Equipment2Running AND NOT Equipment3Running;
AllowStart := AllStopped;

PROGRAM MAIN
VAR
    Interlock : MotorInterlock;
    StartButton : BOOL;
    MotorRunning : BOOL;

    // Equipment status inputs
    Eq1_Status : BOOL := FALSE;
    Eq2_Status : BOOL := FALSE;
    Eq3_Status : BOOL := FALSE;
END_VAR

// Link inputs to the function block
Interlock.Equipment1Running := Eq1_Status;
Interlock.Equipment2Running := Eq2_Status;
Interlock.Equipment3Running := Eq3_Status;

// Motor start logic with interlock
IF StartButton AND Interlock.AllowStart THEN
    MotorRunning := TRUE;
END_IF;
