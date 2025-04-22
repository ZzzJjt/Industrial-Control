**Empty Bottle Removal for Packaging Line Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text (ST) program to automate the removal of empty bottles in a packaging line. After bottles are filled, they are transported by a conveyor toward the packaging station. The system includes two proximity sensors: one detects the presence of any bottle, and the second detects only empty bottles. When an empty bottle is detected, a pneumatic cylinder is activated to remove the empty bottle from the conveyor before it reaches the packaging area.

Ensure that the program controls the conveyor and cylinder operations efficiently, preventing any empty bottles from continuing to the packaging process, while maintaining smooth operation for filled bottles.

**R-I-S-E:**

🟥 R (Role) – Your Role

You are a PLC automation engineer responsible for designing a Structured Text program (IEC 61131-3) that automates the removal of empty bottles from a high-speed packaging line using proximity sensors and a pneumatic ejection system.

⸻

🟩 I (Input) – What You’re Given

You are working with the following hardware and signals:
	•	Sensors:
	•	BottlePresentSensor : BOOL — detects any bottle on the conveyor
	•	EmptyBottleSensor : BOOL — detects whether a bottle is empty
	•	Actuators:
	•	ConveyorMotor : BOOL — drives the conveyor belt
	•	EjectCylinder : BOOL — activates a pneumatic cylinder to remove empty bottles
	•	Timing Device:
	•	EjectTimer : TON — controls the ejection duration (e.g., 0.5 seconds)

⸻

🟧 S (Steps) – What You Need to Do
	1.	Keep the conveyor running continuously to transport bottles toward the packaging station.
	2.	Detect empty bottles:
	•	If both BottlePresentSensor and EmptyBottleSensor are TRUE, the system identifies an empty bottle on the line.
	3.	Activate the ejection system:
	•	Trigger the EjectCylinder to remove the empty bottle.
	•	Use EjectTimer to control how long the cylinder stays extended (e.g., 500 ms).
	4.	Reset the system after ejection:
	•	Once EjectTimer.Q = TRUE, retract the cylinder by setting EjectCylinder := FALSE.
	5.	Ignore filled bottles:
	•	If BottlePresentSensor is TRUE but EmptyBottleSensor is FALSE, allow the bottle to continue without interference.

⸻

🟦 E (Expectation) – What Success Looks Like

You will deliver a modular, real-time ST program that:
	•	Automatically detects and ejects empty bottles without stopping the conveyor
	•	Maintains continuous flow of filled bottles toward the packaging station
	•	Uses timed control for the pneumatic cylinder to reduce wear and prevent overactuation
	•	Is easy to expand with features like bottle counters, alarms, or HMI integration

⸻

🔧 Example Code Snippet:
VAR
    BottlePresentSensor : BOOL;
    EmptyBottleSensor : BOOL;
    ConveyorMotor : BOOL := TRUE;
    EjectCylinder : BOOL;
    EjectTimer : TON;
END_VAR

// Ejection logic
IF BottlePresentSensor AND EmptyBottleSensor THEN
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
