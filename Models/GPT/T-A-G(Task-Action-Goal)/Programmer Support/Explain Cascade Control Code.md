✅ Primary Loop – Pressure Control (Outer Loop)
	•	Purpose: Maintain stable vessel pressure.
	•	How it works:
	•	Reads the actual vessel pressure:
PV1 := ReadPressure();
	•	Compares it to a fixed pressure setpoint:
SP1 := Desired_Pressure;
	•	Computes error: Error1 := SP1 - PV1
	•	Applies PID (with gains Kp1, Ki1, Kd1):
	•	Integral and derivative terms updated using fixed sample time dt
	•	Output OP1 becomes the setpoint for the inner flow loop:
	•	SP2 := OP1
	•	Clamped to ensure output remains within valid limits (0–100%)

⸻

✅ Secondary Loop – Flow Control (Inner Loop)
	•	Purpose: React quickly to achieve target flow rate (SP2), compensating for disturbances.
	•	How it works:
	•	Reads real-time flow rate:
PV2 := ReadFlowRate();
	•	Compares it to the new flow setpoint (from pressure loop):
SP2 := OP1
	•	Computes error: Error2 := SP2 - PV2
	•	Applies PID control (with Kp2, Ki2, Kd2)
	•	Output OP2 controls actuator directly:
	•	SetValvePosition(OP2);
	•	Again clamped (0–100%) for safety and actuator integrity.

⸻

⏱ Timing Considerations
	•	Both loops use a fixed sample time:
	•	dt := t#100ms
	•	Inner loop (flow) reacts faster and more frequently than the outer (pressure), as is standard in cascade control architecture.

⸻

🎯 What This Cascade Control Achieves
	•	Fast correction of disturbances in flow before they propagate to pressure
	•	Tighter, more stable pressure control by leveraging a responsive inner loop
	•	Robust safety with clamping for both valve and controller outputs
	•	Modular structure, making it ideal for expansion (e.g., SCADA interface, tuning mode)
