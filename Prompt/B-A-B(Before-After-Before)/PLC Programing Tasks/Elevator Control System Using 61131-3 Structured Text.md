**Elevator Control System Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text program to control an elevator in a 5-floor building. Each floor has top and bottom limit switches to detect the elevator’s position. The elevator door should remain open for 7 seconds after reaching a floor. If no buttons inside the elevator cabin are pressed during this time, the door should reopen for an additional 10 seconds before closing. The elevator’s movement is governed by its current direction and the direction imposed by the up and down call buttons on each

**B-A-B:**

🟥 B (Before) – The Challenge

Controlling a multi-floor elevator system requires precise management of position detection, door timing, call requests, and directional logic. Without structured logic, an elevator may respond inefficiently to floor calls, close doors prematurely, or even fail to register idle call states. Implementing this in IEC 61131-3 Structured Text is particularly challenging due to the need for timers, state control, and input prioritization across multiple floors.

⸻

🟩 A (After) – The Ideal Outcome

Write a self-contained IEC 61131-3 Structured Text program to control a 5-floor elevator with the following behavior:
	•	Position Detection:
	•	Each floor has top and bottom limit switches to determine the elevator’s current position.
	•	Door Control:
	•	The door opens upon reaching a floor and stays open for 7 seconds.
	•	If no inside cabin button is pressed during this time, the door reopens for an additional 10 seconds before finally closing.
	•	Movement and Direction:
	•	Elevator direction is governed by:
	•	Current direction of motion (up or down)
	•	Up and down call buttons located on each floor
	•	Inside cabin floor selection buttons
	•	Priority and Logic:
	•	Requests are served in the current direction of travel before reversing.
	•	The system should handle pending calls and idle gracefully.

⸻

🟧 B (Bridge) – The Implementation Strategy

To build the system:
	1.	Declare necessary variables:
	•	Floor call arrays: UpCall[1..4], DownCall[2..5], CabinRequest[1..5]
	•	Position inputs: AtFloor[1..5], TopLimit[1..5], BottomLimit[1..5]
	•	Door control: DoorOpen : BOOL, DoorTimer1 : TON, DoorTimer2 : TON
	•	Direction: GoingUp : BOOL, Idle : BOOL
	2.	Manage floor arrival and door timing:
 IF AtFloor[CurrentFloor] THEN
    DoorOpen := TRUE;
    DoorTimer1(IN := TRUE, PT := T#7s);
    IF DoorTimer1.Q THEN
        IF NOT AnyCabinButtonPressed THEN
            DoorTimer2(IN := TRUE, PT := T#10s);
            IF DoorTimer2.Q THEN
                DoorOpen := FALSE;
            END_IF;
        ELSE
            DoorOpen := FALSE;
        END_IF;
    END_IF;
END_IF;
	3.	Handle movement logic:
	•	If GoingUp = TRUE, service floors above first, using UpCall[] and CabinRequest[]
	•	If no more calls in current direction, switch direction
	•	Maintain priority logic to avoid skipping requests or oscillating between directions
	4.	Ensure safety and defaults:
	•	Prevent motion during door open
	•	Reset timers upon floor changes
	•	Stay idle if no requests are active
