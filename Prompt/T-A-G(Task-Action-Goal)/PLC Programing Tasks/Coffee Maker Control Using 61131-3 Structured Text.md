**Coffee Maker Control Using 61131-3 Structured Text:**

Write a self-contained 61131-3 structured text (ST) program to control a coffee machine that manages three tanks (coffee, milk, and mixer) and three valves (one for coffee, one for milk, and one for output). The machine should mix coffee and milk properly to create the best output, following this process:

System Description:

	1.	Tanks and Valves:
	•	The coffee and milk valves open to fill the mixer tank. The mixer tank can hold up to 130ml, and when it reaches the maximum level, the coffee and milk valves will close.
	2.	Mixing Process:
	•	Once the tank is full, the mixer starts automatically and runs for 4 seconds. After mixing is complete, the output valve opens to dispense the coffee.
	3.	Control Buttons:
	•	Button 1: Emergency Stop — Stops the entire system instantly in case of malfunction, such as valve failures, tank level issues, or mixer failures.
	•	Button 2: Start — Begins the coffee-making process.
	•	Button 3: Coffee and Milk — Prepares coffee with milk by opening both the coffee and milk valves.
	•	Button 4: Coffee Only — Prepares coffee without milk by only opening the coffee valve.

Safety Features:

	•	Implement the emergency stop to handle unexpected events, such as valve malfunctions, tank level detection failures, or mixer operation failures. When triggered, the system should stop immediately to ensure safety.

**T-A-G:**

🟥 T (Task) – What You Need to Do

Develop a self-contained control program using IEC 61131-3 Structured Text (ST) for an automated coffee maker. The system must handle tank filling, mixing, dispensing, and emergency stops, based on multiple input buttons and sensor feedback.

⸻

🟩 A (Action) – How to Do It
	1.	Define system components:
	•	Inputs:
	•	Start, CoffeeMilk, CoffeeOnly, EmergencyStop
	•	MixerLevelFull (sensor indicating mixer tank is full)
	•	Outputs:
	•	CoffeeValve, MilkValve, OutputValve, Mixer
	•	Mixer tank capacity: 130 ml
	•	Mixer run time: 4 seconds (use a TON timer)
	2.	Design the control logic:
	•	Start sequence:
	•	If CoffeeMilk is selected, open both CoffeeValve and MilkValve
	•	If CoffeeOnly is selected, open only CoffeeValve
	•	Filling:
	•	Monitor MixerLevelFull
	•	Once full, close input valves and start Mixer
	•	Mixing:
	•	Run mixer for 4 seconds using a timer
	•	When mixing completes, stop the mixer and open OutputValve to dispense
	•	Emergency stop logic:
	•	At any point, if EmergencyStop is pressed, immediately set all outputs to FALSE and halt the process
	3.	Use a state machine (optional) for clarity and future expansion:
 CASE State OF
    0: // Idle
        IF Start THEN
            IF CoffeeMilk THEN
                CoffeeValve := TRUE;
                MilkValve := TRUE;
            ELSIF CoffeeOnly THEN
                CoffeeValve := TRUE;
            END_IF;
            State := 1;
        END_IF;

    1: // Filling
        IF MixerLevelFull THEN
            CoffeeValve := FALSE;
            MilkValve := FALSE;
            Mixer := TRUE;
            MixTimer(IN:=TRUE, PT:=T#4s);
            State := 2;
        END_IF;

    2: // Mixing
        IF MixTimer.Q THEN
            Mixer := FALSE;
            OutputValve := TRUE;
            State := 3;
        END_IF;
END_CASE;

// Emergency handling
IF EmergencyStop THEN
    CoffeeValve := FALSE;
    MilkValve := FALSE;
    OutputValve := FALSE;
    Mixer := FALSE;
    State := 0;
END_IF;
🟦 G (Goal) – What You Want to Achieve

Deliver a safe, responsive, and user-controlled coffee machine automation logic that:
	•	Supports both drink options: coffee-only and coffee with milk
	•	Ensures accurate mixing and dispensing using timers and level sensors
	•	Handles emergency situations immediately and safely
	•	Is modular, readable, and maintainable, ready for HMI or status display integration
	•	Can be easily expanded with features like cleaning cycles, maintenance modes, or ingredient level checks
