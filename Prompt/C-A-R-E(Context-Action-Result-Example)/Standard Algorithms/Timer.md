**Timer:**
Implement a self-contained function block in IEC 61131-3 structured text to create a timer. Include configurable time duration, start/stop controls, and timeout handling.


**C-A-R-E:**

🟥 C (Context) – Background Situation

Timers are fundamental building blocks in PLC programming, used for delays, sequencing, and interlock conditions. However, many standard timers are rigid or lack modular control logic, making it difficult to customize behavior such as dynamically changing durations or adding start/stop logic. For reusable automation code, a flexible and configurable Structured Text timer block is preferred.

⸻

🟩 A (Action) – Task to Perform

Design a self-contained timer function block in IEC 61131-3 Structured Text that includes:
	•	Configurable time duration via PresetTime : TIME
	•	Start and Stop inputs to control execution dynamically
	•	Elapsed time tracking with a built-in timeout mechanism
	•	Internal logic to handle rising-edge detection and accurate scan-cycle timing
	•	Output flags including:
	•	Running : BOOL – TRUE when counting
	•	Done : BOOL – TRUE when the timer completes
	•	Optionally, RemainingTime : TIME

⸻

🟨 R (Result) – Expected Outcome

The outcome will be a modular, reusable timer block that:
	•	Integrates cleanly into any PLC program requiring timing functionality
	•	Enables dynamic control with start/stop inputs
	•	Offers scan-cycle accuracy and proper handling of timer overflow
	•	Can be used across a wide range of automation tasks (e.g., heating cycles, alarms, delays)

⸻

🟦 E (Example) – Concrete Illustration

Simplified logic:

IF Start AND NOT Stop THEN
    IF NOT Running THEN
        StartTime := TIME(); // Capture current time
        Running := TRUE;
    END_IF;

    ElapsedTime := TIME() - StartTime;

    IF ElapsedTime >= PresetTime THEN
        Done := TRUE;
        Running := FALSE;
    END_IF;
ELSIF Stop THEN
    Running := FALSE;
    Done := FALSE;
    ElapsedTime := T#0s;
END_IF;

