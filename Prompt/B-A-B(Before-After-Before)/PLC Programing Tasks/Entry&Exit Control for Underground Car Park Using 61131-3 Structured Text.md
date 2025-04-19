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


**B-A-B:**

🟥 B (Before) – The Challenge

Managing vehicle flow in a single-lane underground car park requires precise coordination to prevent collisions in the narrow passage. Without intelligent control, vehicles entering from both ends simultaneously could lead to dangerous situations. A PLC-controlled system using IEC 61131-3 Structured Text must monitor entry sensors and manage red/green traffic lights to allow only one vehicle in the passage at a time.

⸻

🟩 A (After) – The Ideal Outcome

You will create a Structured Text program that:
	•	Monitors photoelectric sensors (X1, X2) and scan-cycle flags (M1–M4) to detect car movement from either direction.
	•	Sets intermediate flags (M20 for ground floor, M30 for basement) to represent that a car is currently using the passage.
	•	Automatically switches lights:
	•	Red lights (Y1) turn ON when a car is in the passage, blocking both sides.
	•	Green lights (Y2) remain ON only when the passage is empty, indicating it’s safe to enter or exit.
	•	Initial state: Green lights ON, red lights OFF — indicating the passage is free.

⸻

🟧 B (Bridge) – The Implementation Strategy

To achieve this, follow these steps in your program logic:
1. Declare variables:
   VAR
    X1, X2 : BOOL; // Photoelectric sensors
    M1, M2, M3, M4 : BOOL; // Scan pulse flags
    M20, M30 : BOOL; // Passage occupancy from ground or basement
    Y1, Y2 : BOOL; // Outputs for red and green lights
END_VAR
2. Detect entry into passage:
   // Set occupancy flags when a car enters the passage
IF M1 OR M4 THEN
    M20 := TRUE; // Car entering from ground floor
END_IF;

IF M2 OR M3 THEN
    M30 := TRUE; // Car entering from basement
END_IF;
3. Clear occupancy flags when car exits:
// Clear flags when car leaves the passage
IF M3 OR M4 THEN
    M20 := FALSE;
END_IF;

IF M1 OR M2 THEN
    M30 := FALSE;
END_IF;
4. Control the lights:
// If the passage is occupied, turn red lights ON, green lights OFF
IF M20 OR M30 THEN
    Y1 := TRUE;  // Red ON
    Y2 := FALSE; // Green OFF
ELSE
    Y1 := FALSE; // Red OFF
    Y2 := TRUE;  // Green ON
END_IF;
