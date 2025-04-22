**Elevator Control System Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text program to control an elevator in a 5-floor building. Each floor has top and bottom limit switches to detect the elevator’s position. The elevator door should remain open for 7 seconds after reaching a floor. If no buttons inside the elevator cabin are pressed during this time, the door should reopen for an additional 10 seconds before closing. The elevator’s movement is governed by its current direction and the direction imposed by the up and down call buttons on each

**C-A-R-E:**

🟥 C (Context) – The Background

Elevator systems in multi-floor buildings require precise coordination of motion control, door timing, and call handling logic to ensure safe, efficient, and user-friendly operation. In industrial automation, implementing such logic using IEC 61131-3 Structured Text (ST) is common, but it demands clear handling of state transitions, timers, and request queues for both cabin and floor buttons.

⸻

🟩 A (Action) – The Implementation Task

Develop a self-contained Structured Text program to control a 5-floor elevator system with the following behavior:
	•	Position Detection:
	•	Each floor has top and bottom limit switches or position flags (AtFloor[1..5]).
	•	Door Logic:
	•	When the elevator reaches a floor, the door opens and stays open for 7 seconds.
	•	If no button is pressed inside the cabin during this time, the door reopens for an additional 10 seconds before closing.
	•	Call Handling:
	•	Handle calls from up/down buttons on each floor (UpCall[1..4], DownCall[2..5])
	•	Handle cabin requests (CabinRequest[1..5])
	•	Direction Control:
	•	The elevator prioritizes servicing requests in the current direction.
	•	When no further requests exist in the current direction, it reverses.
	•	Safety Logic:
	•	The elevator does not move while the door is open.
	•	All pending requests must be processed before going idle.

⸻

🟨 R (Result) – The Expected Outcome

The final program should:
	•	Accurately manage elevator movement between floors
	•	Open the door automatically for 7 seconds, and extend to 10 more seconds if no cabin request is made
	•	Handle up/down calls and cabin requests intelligently based on direction
	•	Avoid skipping floor requests and handle direction changes safely
	•	Be fully modular, clear, and PLC-deployable following IEC 61131-3 standards

⸻

🟦 E (Example) – A Practical Code Snippet
IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);

    IF DoorTimer1.Q THEN
        IF NOT AnyCabinRequest THEN
            DoorTimer2(IN := TRUE, PT := T#10s);
            IF DoorTimer2.Q THEN
                DoorOpen := FALSE;
            END_IF;
        ELSE
            DoorOpen := FALSE;
        END_IF;
    END_IF;
END_IF;

// Prevent motion while door is open
IF NOT DoorOpen THEN
    IF GoingUp THEN
        // Service next request above
    ELSIF GoingDown THEN
        // Service next request below
    END_IF;
END_IF;
