**Interlock Pressure Relief:**

Develop a self-contained IEC 61131-3 Structured Text program to implement an interlock system for opening a pressure relief valve in a vessel upon detecting overpressure. The program should monitor the vessel pressure using a pressure sensor and trigger the opening of the relief valve when the pressure exceeds a predefined safe limit.

The logic should ensure that the pressure relief valve remains open until the vessel pressure drops below a safe threshold. Additionally, incorporate safety checks to account for sensor failures or valve malfunctions, ensuring that the system defaults to a safe state in the event of a fault. Discuss the significance of pressure relief systems in protecting industrial processes from overpressure hazards and ensuring operational safety.

**R-T-F:**

🟥 R (Role) – Your Role

You are an automation engineer responsible for developing a pressure relief interlock system using IEC 61131-3 Structured Text to ensure vessel safety in an industrial process environment.

⸻

🟩 T (Task) – What You Need to Do

Create a self-contained Structured Text program that automatically opens a pressure relief valve when the vessel’s pressure exceeds a safe limit. The system should:
	•	Monitor pressure via a pressure transmitter (PT)
	•	Open the relief valve if pressure exceeds a defined high limit (e.g., 15 bar)
	•	Keep the valve open until pressure drops below a reset threshold (e.g., 12 bar)
	•	Include fail-safe conditions to:
	•	Open the valve if the sensor fails or provides an invalid signal
	•	Open the valve or raise an alert if the valve itself fails to actuate
	•	Prevent automatic closure of the valve until conditions are verified to be safe

Also, briefly explain how such interlocks are critical for preventing overpressure incidents, protecting equipment, and ensuring operational safety.

⸻

🟧 F (Format) – Expected Output

Provide the following:
	•	A well-structured IEC 61131-3 ST code block with:
	•	Pressure monitoring
	•	Threshold-based valve control
	•	Fault detection logic
	•	Latching behavior for controlled reset
	•	Clear variable declarations (e.g., PT_101, RELIEF_VALVE, SensorFault, ValveFault)
	•	Inline comments for clarity and maintenance
	•	A short explanation of the safety benefits of automated pressure relief interlocks
