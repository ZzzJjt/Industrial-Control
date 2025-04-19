**Entry/Exit Control for Underground Car Park Using 61131-3 Structured Text:**

Write a PLC program in structured text (ST) according to IEC 61131-3 to control the entry and exit of an underground car park. The system uses the following sensors and actuators:

	•	Sensors:
	•	X1: Photoelectric switch at the ground floor entry/exit. It will be ON when a car passes.
	•	X2: Photoelectric switch at the basement entry/exit. It will be ON when a car passes.
	•	M1: ON for one scan cycle when a car from the ground floor passes X1.
	•	M2: ON for one scan cycle when a car from the basement passes X1.
	•	M3: ON for one scan cycle when a car from the basement passes X2.
	•	M4: ON for one scan cycle when a car from the ground floor passes X2.
	•	Intermediate Variables:
	•	M20: ON during the process of a car entering the passage from the ground floor.
	•	M30: ON during the process of a car entering the passage from the basement.
	•	Output Devices:
	•	Y1: Red lights at the entry/exit of the ground floor and the basement.
	•	Y2: Green lights at the entry/exit of the ground floor and the basement.

Process Description:

The entry and exit of the underground car park is controlled by a single lane passage, with traffic lights regulating car movement. The red lights (Y1) prohibit cars from entering or leaving, while the green lights (Y2) allow movement.

	•	When a car enters the passage from the ground floor entry, the red lights at both the ground floor and basement turn ON, while the green lights turn OFF, preventing any other cars from entering or leaving until the car passes through the passage.
	•	Similarly, when a car enters from the basement, the red lights will turn ON at both entry points, prohibiting other vehicles from entering or leaving until the car passes through.
	•	When the passage is clear, the green lights will turn ON again, allowing cars to enter or exit freely.
	•	Initially, the PLC should set the green lights ON and the red lights OFF to indicate free movement.

**T-A-G:**

🟥 T (Task) – What You Need to Do

Develop a Structured Text (ST) program using IEC 61131-3 to control entry and exit traffic lights for a single-lane underground car park shared by the ground floor and basement. The system must prevent cars from entering the passage at both ends simultaneously by coordinating sensor inputs and traffic light outputs.

⸻

🟩 A (Action) – How to Do It
	1.	Monitor Sensor and Memory Flags:
	•	X1: Detects car passage at the ground floor
	•	X2: Detects car passage at the basement
	•	M1–M4: One-scan memory flags indicating car direction through X1 or X2
	•	M20: Set when a car is in the passage from the ground floor
	•	M30: Set when a car is in the passage from the basement
	2.	Control Lights Based on Passage Occupancy:
	•	If M1 or M4 is TRUE → M20 := TRUE (car from ground floor)
	•	If M2 or M3 is TRUE → M30 := TRUE (car from basement)
	•	If M3 or M4 is TRUE → M20 := FALSE (car from ground floor has exited)
	•	If M1 or M2 is TRUE → M30 := FALSE (car from basement has exited)
	3.	Update Output Lights:
	•	If M20 or M30 is TRUE:
	•	Y1 := TRUE (red light ON), Y2 := FALSE (green light OFF)
	•	If both are FALSE (no cars in passage):
	•	Y1 := FALSE, Y2 := TRUE (allow traffic)
	4.	Initialize System:
	•	On power-up, allow free movement:
	•	Y1 := FALSE, Y2 := TRUE

⸻

🟦 G (Goal) – What You Want to Achieve

Deliver a reliable traffic light control program that:
	•	Ensures only one vehicle uses the passage at a time
	•	Prevents simultaneous entry from both directions
	•	Automatically resets lights once the passage is clear
	•	Starts with green lights enabled for free traffic flow
	•	Is suitable for real-time deployment in a shared underground parking system
