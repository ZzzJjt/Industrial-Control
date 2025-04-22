**Interlock Overfill Protection:**

Develop a self-contained IEC 61131-3 Structured Text program to implement an interlock system for overfill protection of a vessel. The program should utilize a level sensor to monitor the liquid level in the vessel and control an inlet valve to prevent overfilling.

The logic should ensure that the inlet valve automatically closes when the level sensor detects that the liquid has reached a predefined high-level setpoint. The interlock should remain engaged until the level drops below a safe threshold. Additionally, include a fail-safe mechanism to handle sensor failure or valve malfunction, ensuring the system defaults to a safe state. Discuss the importance of interlocks in industrial process safety and the role of overfill protection in preventing hazardous conditions.

**C-A-R-E:**

🟥 C (Context) – The Background

In industrial process environments, uncontrolled filling of tanks or vessels can lead to overflows, environmental hazards, equipment damage, and safety violations. Overfill incidents are preventable using automated interlock systems that monitor fluid levels and shut off inlet sources in time. Integrating overfill protection into PLC-based control systems ensures consistent, repeatable, and fail-safe operation.

⸻

🟩 A (Action) – The Implementation Task

Develop a self-contained IEC 61131-3 Structured Text program that implements an overfill protection interlock for a vessel. The system must:
	•	Use a level sensor to monitor liquid level in real-time
	•	Automatically close the inlet valve when the level reaches a defined high-level setpoint
	•	Keep the valve closed until the level drops below a lower reset threshold, ensuring the interlock is not prematurely released
	•	Include fail-safe logic to close the inlet valve in case of:
	•	Sensor failure (e.g., stuck value or offline condition)
	•	Valve malfunction (e.g., failure to respond to control signal)

The interlock must maintain a latched shutdown state and require proper level conditions to clear.

⸻

🟨 R (Result) – The Expected Outcome

The resulting interlock logic ensures reliable overfill protection, automatically preventing the tank from exceeding its safe capacity. It minimizes risk of spills, maintains process stability, and ensures that system behavior is predictable even in fault conditions. It also supports industry best practices for safety and environmental compliance.

⸻

🟦 E (Example) – A Practical Use Case

When the level sensor detects that the tank level exceeds 95% capacity, the system sets a SHUTDOWN_LATCH := TRUE and closes the inlet valve by setting INLET_VALVE := FALSE. If the sensor fails or its signal is outside valid bounds, the system immediately defaults to a safe state by closing the inlet valve. The valve remains closed until the level drops below 85%, clearing the latch and allowing normal operation to resume. This logic is essential in industries such as chemical processing, water treatment, and oil & gas.
