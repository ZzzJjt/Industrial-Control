**Automatic Car Wash Station Control Using 61131-3 Structured Text:**

Write a PLC program in structured text (ST) according to IEC 61131-3 standards to control an automatic car wash station with a single wash bay. The system should use a sensor to detect the presence of a car. Once the car is detected and no human is present in the wash area, the system should initiate the car wash process. If a human is detected within the car wash area at any point, the station should immediately stop the wash process, trigger an alarm, and remain in a safe state until the area is clear.

**C-A-R-E:**

🟥 C (Context) – The Background

An automatic car wash station must operate efficiently while ensuring safety compliance, particularly when human presence is detected within the wash bay. PLCs programmed using IEC 61131-3 Structured Text (ST) are ideal for implementing reliable, real-time control logic. Without proper interlocks and state management, the wash process could pose serious safety risks if activated while a person is inside the wash area.

⸻

🟩 A (Action) – The Implementation Task

Develop a PLC program in Structured Text (ST) that meets the following criteria:
	•	Inputs:
	•	CarPresentSensor : BOOL — detects the presence of a vehicle
	•	HumanDetectedSensor : BOOL — detects a person in the wash area
	•	Outputs:
	•	WashActive : BOOL — controls wash cycle activation
	•	AlarmActive : BOOL — triggers safety alarm
	•	SafeToRun : BOOL — governs whether the wash cycle can be safely initiated
	•	Control Logic:
	•	Start the wash process only if:
	•	A car is present and
	•	No human is detected and
	•	The system is in a safe-to-run state
	•	Immediately stop the wash, set AlarmActive := TRUE, and block any further operation if a person is detected
	•	Allow operation to resume only after the wash is off and the area is clear

⸻

🟨 R (Result) – The Expected Outcome

The resulting PLC logic will:
	•	Prevent any unsafe operation of the car wash
	•	React immediately to the presence of a human by halting the system and raising an alarm
	•	Stay in a safe interlocked state until the area is verified to be clear
	•	Provide clear outputs (WashActive, AlarmActive, SafeToRun) that can be connected to actuators, HMI displays, or safety relays
	•	Be written in a modular and maintainable format suitable for further expansion (e.g., timed wash stages)

⸻

🟦 E (Example) – A Practical Code Snippet
CarPresent := CarPresentSensor;
HumanDetected := HumanDetectedSensor;

IF HumanDetected THEN
    WashActive := FALSE;
    AlarmActive := TRUE;
    SafeToRun := FALSE;
ELSIF CarPresent AND NOT HumanDetected AND SafeToRun THEN
    WashActive := TRUE;
    AlarmActive := FALSE;
END_IF;

// Allow system to reset once area is clear and wash has stopped
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF;
This code ensures failsafe behavior while allowing automatic operation to resume only under controlled conditions. It forms the basis of a scalable, standards-compliant solution for a single-bay car wash system.
