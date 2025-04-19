**Empty Bottle Removal for Packaging Line Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text (ST) program to automate the removal of empty bottles in a packaging line. After bottles are filled, they are transported by a conveyor toward the packaging station. The system includes two proximity sensors: one detects the presence of any bottle, and the second detects only empty bottles. When an empty bottle is detected, a pneumatic cylinder is activated to remove the empty bottle from the conveyor before it reaches the packaging area.

Ensure that the program controls the conveyor and cylinder operations efficiently, preventing any empty bottles from continuing to the packaging process, while maintaining smooth operation for filled bottles.

**C-A-R-E:**

🟥 C (Context) – The Background

In a packaging line, it’s essential to ensure that only filled bottles proceed to the packaging station. Any empty bottles that slip through can cause product defects, waste materials, or disrupt the downstream packaging process. To automate this task, a PLC-based system using IEC 61131-3 Structured Text (ST) can manage the removal of empty bottles using proximity sensors and a pneumatic ejector.

⸻

🟩 A (Action) – The Implementation Task

Develop a self-contained Structured Text program that automates the removal of empty bottles using:
	•	Two proximity sensors:
	•	BottlePresentSensor: detects the presence of a bottle
	•	EmptyBottleSensor: detects whether the bottle is empty
	•	One pneumatic cylinder:
	•	EjectCylinder: pushes empty bottles off the conveyor
	•	One conveyor motor:
	•	ConveyorMotor: keeps bottles moving toward the packaging station

Operational Logic:
	•	The conveyor runs continuously.
	•	If both sensors detect an empty bottle, activate the ejector for a brief time (e.g., 0.5 seconds).
	•	Filled bottles continue through the line uninterrupted.
	•	Use a timer to ensure the cylinder retracts after each ejection.

⸻

🟨 R (Result) – The Expected Outcome

The result is a real-time, efficient control system that:
	•	Removes only empty bottles, avoiding disruptions in the packaging process
	•	Ensures filled bottles proceed smoothly, improving production throughput
	•	Uses a timer-based ejector pulse, avoiding mechanical wear from constant actuation
	•	Provides a scalable and modifiable control foundation for future enhancements (e.g., alarm logic or reject counters)

⸻

🟦 E (Example) – A Practical Code Snippet

VAR
    BottlePresentSensor : BOOL;
    EmptyBottleSensor : BOOL;
    ConveyorMotor : BOOL := TRUE; // Always running
    EjectCylinder : BOOL;
    EjectTimer : TON;
END_VAR

// Eject logic for empty bottles
IF BottlePresentSensor AND EmptyBottleSensor THEN
    EjectTimer(IN := TRUE, PT := T#500ms); // Activate ejector for 0.5s
    EjectCylinder := TRUE;
ELSE
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
