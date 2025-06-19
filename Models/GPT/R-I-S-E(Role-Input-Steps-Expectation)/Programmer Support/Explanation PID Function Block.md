VAR
    PID_CTRL : PID; // OSCAT PID block instance

    SP       : REAL := 75.0;  // Setpoint (e.g., 75°C)
    PV       : REAL;          // Measured value from sensor

    Kp       : REAL := 2.0;   // Proportional gain
    Ki       : REAL := 0.5;   // Integral gain
    Kd       : REAL := 0.1;   // Derivative gain

    MANUAL   : BOOL := FALSE; // Manual override flag
    MV_MAN   : REAL := 0.0;   // Manual output value

    LIMIT_HI : REAL := 100.0; // Max output
    LIMIT_LO : REAL := 0.0;   // Min output

    OUT      : REAL;          // PID output
    ERROR    : REAL;          // PID error
    SAT      : BOOL;          // Optional: output is saturated
END_VAR


// Assign input values to PID block
PID_CTRL.SP        := SP;
PID_CTRL.PV        := PV;
PID_CTRL.KP        := Kp;
PID_CTRL.KI        := Ki;
PID_CTRL.KD        := Kd;
PID_CTRL.MANUAL    := MANUAL;
PID_CTRL.MV_MAN    := MV_MAN;
PID_CTRL.LIMIT_HI  := LIMIT_HI;
PID_CTRL.LIMIT_LO  := LIMIT_LO;

// Execute PID block
PID_CTRL();  // Function block call

// Extract outputs
OUT   := PID_CTRL.OUT;
ERROR := PID_CTRL.ERROR;
SAT   := PID_CTRL.SAT;
