**Empty Bottle Removal for Packaging Line Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text (ST) program to automate the removal of empty bottles in a packaging line. After bottles are filled, they are transported by a conveyor toward the packaging station. The system includes two proximity sensors: one detects the presence of any bottle, and the second detects only empty bottles. When an empty bottle is detected, a pneumatic cylinder is activated to remove the empty bottle from the conveyor before it reaches the packaging area.

Ensure that the program controls the conveyor and cylinder operations efficiently, preventing any empty bottles from continuing to the packaging process, while maintaining smooth operation for filled bottles.

**B-A-B:**

🟥 B (Before) – The Challenge

In high-speed packaging lines, empty bottles must be removed before packaging to ensure product integrity and avoid packaging errors. Manual inspection or poorly timed automation can lead to misplaced bottles, line jams, or damaged packaging equipment. Without a reliable detection and ejection system, efficiency and product quality suffer.

⸻

🟩 A (After) – The Ideal Outcome

Create a self-contained IEC 61131-3 Structured Text program that:
	•	Automates the detection and removal of empty bottles using two proximity sensors:
	•	BottlePresentSensor – detects any bottle on the conveyor
	•	EmptyBottleSensor – detects specifically empty bottles
	•	Controls the conveyor to continuously move bottles toward the packaging station
	•	Activates a pneumatic cylinder to eject empty bottles only when detected, ensuring they don’t enter the packaging area
	•	Maintains seamless flow for correctly filled bottles and prevents disruptions in the packaging line
	•	Operates reliably in real-time conditions, coordinating between sensors and actuator timing

⸻

🟧 B (Bridge) – The Implementation Strategy

To implement this logic, you can structure your program as follows:
	1.	Declare variables:
 VAR
    BottlePresentSensor : BOOL;
    EmptyBottleSensor : BOOL;
    ConveyorMotor : BOOL;
    EjectCylinder : BOOL;
    EjectTimer : TON;
END_VAR
	2.	Write logic for bottle detection and ejection:
 // Conveyor always runs
ConveyorMotor := TRUE;

// Trigger ejection if an empty bottle is detected
IF BottlePresentSensor AND EmptyBottleSensor THEN
    EjectTimer(IN := TRUE, PT := T#500ms); // extend cylinder for 0.5s
    EjectCylinder := TRUE;
ELSE
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
	3.	Document behaviors and safety:
	•	Ejection is triggered only when both sensors are TRUE
	•	EjectTimer prevents prolonged or repeated cylinder activation
	•	Filled bottles continue without interruption, preserving production speed
