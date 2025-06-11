FUNCTION_BLOCK FB_MotorInterlock
VAR_INPUT
    // Equipment status inputs – TRUE = Running
    Equipment1Running: BOOL := FALSE;
    Equipment2Running: BOOL := FALSE;
    Equipment3Running: BOOL := FALSE;
END_VAR

VAR_OUTPUT
    // Output – TRUE if it's safe to start the motor
    AllowStart: BOOL := FALSE;
END_VAR

// Allow motor start only if ALL equipment is NOT running
AllowStart :=
    NOT Equipment1Running AND
    NOT Equipment2Running AND
    NOT Equipment3Running;

    PROGRAM PLC_PRG
VAR
    MotorInterlock: FB_MotorInterlock;

    // Simulated or real-world equipment status
    PumplineRunning: BOOL := FALSE;     // e.g., from pump feedback
    ConveyorRunning: BOOL := FALSE;     // e.g., from VFD run signal
    AgitatorRunning: BOOL := FALSE;     // e.g., from motor contactor DI

    // Output to motor start circuit or HMI enable
    bMotorStartAllowed: BOOL := FALSE;
END_VAR

// Call the interlock function block with real-world inputs
MotorInterlock(
    Equipment1Running := PumplineRunning,
    Equipment2Running := ConveyorRunning,
    Equipment3Running := AgitatorRunning
);

// Assign output to internal or external logic
bMotorStartAllowed := MotorInterlock.AllowStart;
