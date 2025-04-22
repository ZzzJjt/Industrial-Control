**Empty Bottle Removal for Packaging Line Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text (ST) program to automate the removal of empty bottles in a packaging line. After bottles are filled, they are transported by a conveyor toward the packaging station. The system includes two proximity sensors: one detects the presence of any bottle, and the second detects only empty bottles. When an empty bottle is detected, a pneumatic cylinder is activated to remove the empty bottle from the conveyor before it reaches the packaging area.

Ensure that the program controls the conveyor and cylinder operations efficiently, preventing any empty bottles from continuing to the packaging process, while maintaining smooth operation for filled bottles.

**T-A-G:**
🟥 T (Task) – What You Need to Do

Develop a self-contained IEC 61131-3 Structured Text (ST) program to automate the removal of empty bottles from a packaging line. The system must use two proximity sensors and a pneumatic cylinder to ensure only filled bottles proceed to the packaging station.

⸻

🟩 A (Action) – How to Do It
	1.	Sensor Logic:
	•	Use BottlePresentSensor : BOOL to detect any bottle on the conveyor.
	•	Use EmptyBottleSensor : BOOL to determine if the detected bottle is empty.
	2.	Conveyor Control:
	•	Set ConveyorMotor := TRUE to run continuously and transport bottles toward the packaging station.
	3.	Ejection Mechanism:
	•	When both sensors are TRUE (an empty bottle is detected), activate the EjectCylinder using a TON timer (e.g., 500 ms).
	•	The cylinder ejects the empty bottle before it reaches the packaging zone.
	•	After the timer finishes, deactivate the cylinder to retract it.
	4.	Code Snippet:
 VAR
    BottlePresentSensor : BOOL;
    EmptyBottleSensor : BOOL;
    ConveyorMotor : BOOL := TRUE;
    EjectCylinder : BOOL;
    EjectTimer : TON;
END_VAR

// Conveyor runs continuously
ConveyorMotor := TRUE;

// Empty bottle detection and ejection logic
IF BottlePresentSensor AND EmptyBottleSensor THEN
    EjectTimer(IN := TRUE, PT := T#500ms);
    EjectCylinder := TRUE;
ELSE
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
🟦 G (Goal) – What You Want to Achieve

Create an efficient and reliable bottle rejection system that:
	•	Automatically detects and removes empty bottles in real time
	•	Keeps the conveyor operating continuously, minimizing downtime
	•	Prevents empty bottles from reaching the packaging area
	•	Ensures filled bottles pass through the system without interruption
	•	Is structured, scalable, and easily integrable into larger automation systems
