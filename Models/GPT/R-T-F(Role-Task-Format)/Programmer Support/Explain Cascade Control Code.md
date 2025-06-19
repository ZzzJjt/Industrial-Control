e1 := SP1 - PV1;                          // Pressure error
e1_sum := e1_sum + e1 * dt;              // Integral of pressure error
e1_diff := (e1 - e1_prev) / dt;          // Derivative of pressure error

OP1 := Kp1 * e1 + Ki1 * e1_sum + Kd1 * e1_diff;  // PID output for pressure
OP1 := LIMIT(OP1, 0.0, 100.0);           // Clamp output to safe range (0–100%)

SP2 := OP1;                              // Setpoint for flow loop from pressure controller
e2 := SP2 - PV2;                         // Flow error
e2_sum := e2_sum + e2 * dt;              // Integral of flow error
e2_diff := (e2 - e2_prev) / dt;          // Derivative of flow error

OP2 := Kp2 * e2 + Ki2 * e2_sum + Kd2 * e2_diff;  // PID output for flow
OP2 := LIMIT(OP2, 0.0, 100.0);           // Clamp output to safe range

SetValvePosition(OP2);                   // Send final output to control valve
