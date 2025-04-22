**Interlock Gas Wellhead:**

Develop a self-contained IEC 61131-3 Structured Text program to implement emergency interlocks for a subsea gas wellhead. The program should monitor critical parameters such as wellhead pressure, temperature, and flow rates using pressure transmitters (PT), temperature transmitters (TT), and flow meters (FT). The interlock system should immediately trigger emergency shutdown procedures if any of these parameters exceed predefined safety limits.

The program should include logic for closing the master valve (MV-101) if wellhead pressure exceeds 1500 psi, or if flow rate drops below the minimum threshold, indicating a potential leak. Additionally, integrate temperature monitoring to shut down the system if the temperature exceeds 120°C. Incorporate safety features such as automatic reset prevention, ensuring that manual intervention is required to restart the system after an emergency shutdown.

Discuss the critical role of emergency interlocks in subsea gas wellhead operations, particularly in preventing catastrophic failures due to pressure, temperature, or flow anomalies.

**R-T-F:**

🟥 R (Role) – Your Role

Act as a PLC programmer responsible for designing and implementing emergency interlock logic for a subsea gas wellhead using IEC 61131-3 Structured Text (ST).

⸻

🟩 T (Task) – What You Need to Do

Develop a self-contained Structured Text program that monitors key process parameters—pressure, temperature, and flow rate—and performs emergency shutdown actions when any parameter exceeds its predefined safety limit. The program should:
	•	Monitor:
	•	PT sensors for pressure
	•	TT sensors for temperature
	•	FT sensors for flow rate
	•	Perform the following actions:
	•	Close master valve (MV-101) if pressure exceeds 1500 psi
	•	Close MV-101 if flow rate drops below a safe minimum threshold (indicating a leak)
	•	Initiate system shutdown if temperature exceeds 120°C
	•	Include logic to latch the shutdown condition and prevent automatic restart
	•	Require manual reset to clear the shutdown flag and restore normal operation

⸻

🟧 F (Format) – Expected Output

Provide a deliverable that includes:
	•	A complete IEC 61131-3 ST program implementing the described interlocks
	•	Clear variable declarations (e.g., PT_101, TT_101, FT_101, MV_101, SHUTDOWN, RESET)
	•	Proper use of IF conditions and latching logic for safe and predictable behavior
	•	A brief explanation of how the interlock system protects the subsea wellhead from catastrophic failures (e.g., blowouts, overheating, or leaks)
