PROGRAM PLC_PRG
VAR
    pid_inst : PID; // Create instance of PID function block
    pv_value : REAL := ReadTemperature(); // Current process variable
    sp_value : REAL := 100.0;             // Target setpoint
    cv_output : REAL;                     // Control output to actuator
END_VAR

// Run the PID block every 100 ms
pid_inst(
    PV := pv_value,
    SP := sp_value,
    KP := 2.5,
    KI := 0.8,
    KD := 0.1,
    DT := T#100MS,
    MIN := 0.0,
    MAX := 100.0,
    CV => cv_output
);

SetValvePosition(cv_output); // Send control signal to actuator
