**Control Narrative for Ammonium Nitrate Reactor**

Create a control narrative for a reactor producing ammonium nitrates including concrete setpoints and ranges.

**T-A-G:**

🟥 T (Task)

Develop a control narrative for a reactor producing ammonium nitrate, focusing on automation-ready descriptions that include concrete process setpoints, equipment, and operating logic.

⸻

🟩 A (Action)
	•	Define the process context: continuous neutralization of nitric acid with ammonia in a stirred reactor
	•	Include concrete setpoints and ranges for critical parameters:
	•	Temperature: 175 °C ± 2 °C
	•	Pressure: 4.8 bar ± 0.2 bar
	•	Ammonia-to-acid molar ratio: 1.01:1
	•	pH range: 6.2 ± 0.3
	•	Specify the instrumentation and control elements:
	•	TIC-101 (reactor temperature controller), PIC-102 (pressure), FIC-103/104 (flow control), AIC-105 (pH)
	•	Outline the operating sequence:
	1.	Startup: Valve alignment, flow ramp-up, temperature control, pH stabilization
	2.	Normal operation: Maintain loop control with alarm monitoring
	3.	Shutdown: Stop feeds, isolate reactor, relieve pressure
	•	Define interlocks and alarm logic:
	•	Trip if pressure > 5.2 bar
	•	Emergency shutdown if temperature > 185 °C
	•	Reduce ammonia feed if pH < 5.5

⸻

🟦 G (Goal)

Produce a clear, structured, and automation-friendly control narrative that supports safe reactor operation, facilitates PLC/DCS implementation, and ensures consistent ammonium nitrate quality through well-defined control logic and interlocks.
