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
