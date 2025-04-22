**Explanation PID Function Block:**
Provide a comprehensive explanation of the inputs and outputs of the PID function block in the OSCAT library, including the role of each parameter in process control and examples of practical implementation in industrial applications.

**R-T-F:**

🟥 R (Role) – Your Role

You are a process automation engineer responsible for configuring and documenting the usage of the PID function block from the OSCAT library. Your objective is to ensure correct tuning, integration, and operation of this block in an industrial PLC environment.

⸻

🟩 T (Task) – What You Need to Do

Explain the inputs and outputs of the PID function block and describe how each parameter contributes to effective process control. You should also include practical usage tips and industrial implementation scenarios.

⸻

🟦 F (Format) – Output Format

🔧 Inputs:
	•	SP: Setpoint – the desired target value (e.g., target temperature or pressure)
	•	PV: Process Variable – the current measured value from the system
	•	Kp, Ki, Kd: PID tuning parameters for proportional, integral, and derivative gains
	•	MANUAL: Optional input to enable manual override
	•	LIMIT_HI, LIMIT_LO: Optional bounds for clamping the control output

⚙️ Internal Logic:
	•	Calculates the error: Error := SP - PV
	•	Uses the error to compute the control signal based on PID formulas
	•	Applies anti-windup and output limiting logic
	•	Supports manual mode if needed for maintenance or fallback control

🔁 Outputs:
	•	OUT: Final control signal (e.g., sent to an actuator, valve, or motor)
	•	ERROR: Live difference between SP and PV, useful for monitoring
	•	Optional: Flags such as LIMIT_ACTIVE, MANUAL_MODE for diagnostics

⸻

✅ Real-World Application Example:

In a tank level control system, use the PID block to control a pump:
SP := 80.0; // Desired level (cm)
PV := ReadLevelSensor(); // Real-time level measurement
Kp := 1.0;
Ki := 0.2;
Kd := 0.05;

OUT := PID(SP, PV, Kp, Ki, Kd); // Control signal sent to pump speed

