VAR
    // Input values
    PV       : REAL := 0.0;       // Measured process variable (e.g., temperature)
    SP       : REAL := 100.0;     // Setpoint (desired value)
    
    // PID tuning parameters
    Kp       : REAL := 2.0;
    Ki       : REAL := 0.5;
    Kd       : REAL := 0.1;

    // PID instance
    pid_ctrl : PID;               // Instance of OSCAT PID function block

    // Time handling
    dt       : TIME := T#100ms;   // Sampling time
    dt_real  : REAL := 0.1;       // Sampling time in seconds (for integration/derivation)

    // Output values
    Output   : REAL;              // Control output (e.g., motor speed or valve position)
END_VAR

// PID Configuration (only once, e.g., on startup)
IF NOT pid_ctrl.init THEN
    pid_ctrl.init := TRUE;
    pid_ctrl.tsam := dt_real;     // Set sampling time
    pid_ctrl.kp := Kp;
    pid_ctrl.ki := Ki;
    pid_ctrl.kd := Kd;
    pid_ctrl.man := FALSE;        // Auto mode (manual override disabled)
    pid_ctrl.min := 0.0;          // Output lower limit
    pid_ctrl.max := 100.0;        // Output upper limit
END_IF;

// Update PID block each cycle
pid_ctrl.sp := SP;               // Update setpoint
pid_ctrl.pv := PV;               // Update process variable
pid_ctrl();                      // Call the PID block

Output := pid_ctrl.out;          // Control output from PID block
