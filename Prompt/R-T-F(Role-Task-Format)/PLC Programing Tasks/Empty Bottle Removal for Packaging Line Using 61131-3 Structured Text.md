**Empty Bottle Removal for Packaging Line Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text (ST) program to automate the removal of empty bottles in a packaging line. After bottles are filled, they are transported by a conveyor toward the packaging station. The system includes two proximity sensors: one detects the presence of any bottle, and the second detects only empty bottles. When an empty bottle is detected, a pneumatic cylinder is activated to remove the empty bottle from the conveyor before it reaches the packaging area.

Ensure that the program controls the conveyor and cylinder operations efficiently, preventing any empty bottles from continuing to the packaging process, while maintaining smooth operation for filled bottles.

**R-T-F:**

🟥 R (Role) – Your Role

You are a PLC programmer responsible for developing a Structured Text (ST) control program for a packaging line. Your objective is to ensure that only filled bottles continue to the packaging station, and that empty bottles are automatically detected and removed without interrupting the flow of production.

⸻

🟩 T (Task) – What You Need to Do

Create a self-contained IEC 61131-3 Structured Text program (not a function block) that:
	1.	Monitors two sensors:
	•	BottlePresentSensor : BOOL — detects the presence of a bottle on the conveyor
	•	EmptyBottleSensor : BOOL — identifies whether the bottle is empty
	2.	Controls two actuators:
	•	ConveyorMotor : BOOL — runs the conveyor continuously
	•	EjectCylinder : BOOL — activates to remove empty bottles
	3.	Implements timed ejection:
	•	When both sensors detect an empty bottle, trigger a timer (e.g., 500 ms)
	•	Extend EjectCylinder := TRUE during this period
	•	Retract (EjectCylinder := FALSE) automatically once the timer completes
	4.	Ensures safety and continuity:
	•	Filled bottles must not be ejected
	•	Conveyor must run continuously and smoothly regardless of ejection events

⸻

🟧 F (Format) – Expected Output

A clear and modular Structured Text implementation, such as:
VAR
    BottlePresentSensor : BOOL;
    EmptyBottleSensor : BOOL;
    ConveyorMotor : BOOL := TRUE;
    EjectCylinder : BOOL;
    EjectTimer : TON;
END_VAR

// Conveyor always runs
ConveyorMotor := TRUE;

// Bottle removal logic
IF BottlePresentSensor AND EmptyBottleSensor THEN
    EjectTimer(IN := TRUE, PT := T#500ms); // Activate ejector for 0.5 seconds
    EjectCylinder := TRUE;
ELSE
    EjectTimer(IN := FALSE);
    IF NOT EjectTimer.Q THEN
        EjectCylinder := FALSE;
    END_IF;
END_IF;
