**Automatic Car Wash Station Control Using 61131-3 Structured Text:**

Write a PLC program in structured text (ST) according to IEC 61131-3 standards to control an automatic car wash station with a single wash bay. The system should use a sensor to detect the presence of a car. Once the car is detected and no human is present in the wash area, the system should initiate the car wash process. If a human is detected within the car wash area at any point, the station should immediately stop the wash process, trigger an alarm, and remain in a safe state until the area is clear.

**B-A-B:**
🟥 B (Before) – The Challenge

In automated environments like car wash stations, ensuring operational safety while maintaining efficiency is critical. Many legacy systems lack reliable interlocks or safety logic to handle dynamic human presence detection. Without proper logic, a car wash system could potentially operate while a human is present in the wash area, posing serious safety hazards and increasing liability risks. PLC-based systems must be able to detect these conditions in real time and respond immediately to prevent accidents.

⸻

🟩 A (After) – The Ideal Outcome

Create a self-contained PLC program using IEC 61131-3 Structured Text (ST) that:
	•	Monitors two key sensors:
	•	CarPresentSensor — detects if a car has entered the wash bay
	•	HumanDetectedSensor — detects if a person is present in the wash zone
	•	Controls the car wash sequence by:
	•	Starting the wash cycle only if:
	•	A car is present AND
	•	No human is detected in the area
	•	Immediately stopping the wash process and triggering an alarm if a human is detected during operation
	•	Holding the system in a safe lockout state until the area is clear again
	•	Follows modular, well-commented logic for each state and transition
	•	Ensures that the system never resumes automatically after a human interruption without confirmation (optional extension)

⸻

🟧 B (Bridge) – The Implementation Strategy

To implement the control logic:
	1.	Declare state variables:
	•	CarPresent : BOOL
	•	HumanDetected : BOOL
	•	WashActive : BOOL
	•	AlarmActive : BOOL
	•	SafeToRun : BOOL := TRUE
	2.	Core logic (Structured Text):
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

// Reset safe state only when area is clear and no wash is running
IF NOT HumanDetected AND NOT WashActive THEN
    SafeToRun := TRUE;
END_IF;
	3.	Add comments to explain:
	•	Why human detection takes priority
	•	How state retention (e.g. SafeToRun) avoids automatic restarts
	•	How to interface this logic with physical actuators and alarms
 
