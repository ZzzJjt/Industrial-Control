**Interlock Overfill Protection:**

Develop a self-contained IEC 61131-3 Structured Text program to implement an interlock system for overfill protection of a vessel. The program should utilize a level sensor to monitor the liquid level in the vessel and control an inlet valve to prevent overfilling.

The logic should ensure that the inlet valve automatically closes when the level sensor detects that the liquid has reached a predefined high-level setpoint. The interlock should remain engaged until the level drops below a safe threshold. Additionally, include a fail-safe mechanism to handle sensor failure or valve malfunction, ensuring the system defaults to a safe state. Discuss the importance of interlocks in industrial process safety and the role of overfill protection in preventing hazardous conditions.

**B-A-B:**

🟥 B (Before) – The Challenge

In many industrial processes, uncontrolled vessel overfilling can lead to dangerous spills, environmental damage, equipment failure, or even fire and explosion hazards. Without an automatic protection mechanism, operators may not react in time to high-level conditions or instrumentation faults. Manual intervention alone is not always reliable, especially in unattended or fast-filling operations.

⸻

🟩 A (After) – The Ideal Outcome

Develop a self-contained IEC 61131-3 Structured Text program that performs automatic overfill protection for a process vessel. The system should:
	•	Continuously monitor the liquid level using a level sensor.
	•	Automatically close the inlet valve (e.g., INLET_VALVE := FALSE) when the level reaches a predefined high-level setpoint.
	•	Keep the valve closed until the level drops below a safe lower reset threshold, avoiding valve chattering.
	•	Include a fail-safe mechanism that:
	•	Closes the inlet valve if the level sensor fails (e.g., stuck signal or sensor offline).
	•	Defaults to a safe state in the event of valve malfunction.

This logic ensures that the system proactively responds to overfill risks and maintains safe operation without requiring manual input under fault conditions.

⸻

🟧 B (Bridge) – The Implementation Strategy

To build this interlock system:
	1.	Define input variables for the level signal, fault detection, and valve status.
	2.	Use conditional logic to monitor for high-level conditions:

 IF LevelSensor > HIGH_SETPOINT THEN
    INLET_VALVE := FALSE;
    SHUTDOWN_LATCH := TRUE;
END_IF;

IF SHUTDOWN_LATCH AND LevelSensor < RESET_THRESHOLD THEN
    SHUTDOWN_LATCH := FALSE;
END_IF;

IF SensorFault OR ValveFault THEN
    INLET_VALVE := FALSE; // Fail-safe action
END_IF;

	3.	Implement latching logic to maintain interlock status until reset conditions are met.
	4.	Include manual override protections only with supervisory control and alarm acknowledgment.

Finally, explain how overfill interlocks are critical to process safety, asset protection, and regulatory compliance, particularly in chemical, pharmaceutical, and oil & gas industries.
