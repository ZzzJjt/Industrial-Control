**Fix Traffic Light Code:**

Can you find the error in the following 61131-3 ST code: PROGRAM TrafficLightControl VAR pedestrianButtonPressed: BOOL; emergencyVehicleApproaching: BOOL; greenLightOn: BOOL; yellowLightOn: BOOL; redLightOn: BOOL; timer: TON; // timer to control the duration of green and yellow lights END_VAR

// Initialize the program greenLightOn := TRUE; yellowLightOn := FALSE; redLightOn := FALSE; timer(IN := NOT emergencyVehicleApproaching, PT := T#5s);

// Main program loop WHILE TRUE DO // Check for pedestrian button press pedestrianButtonPressed := ReadPedestrianButton();

// Check for emergency vehicle approaching
emergencyVehicleApproaching := ReadEmergencyVehicleSensor();

IF emergencyVehicleApproaching THEN
    // Turn off all lights and turn on green light for emergency vehicle
    greenLightOn := TRUE;
    yellowLightOn := FALSE;
    redLightOn := FALSE;
    timer(IN := FALSE);

ELSEIF pedestrianButtonPressed THEN
    // Turn off all lights and turn on red light and pedestrian light
    greenLightOn := FALSE;
    yellowLightOn := FALSE;
    redLightOn := TRUE;
    timer(IN := FALSE);

    // Wait for pedestrian to cross
    WAIT UNTIL NOT pedestrianButtonPressed;

    // Turn on yellow light for warning and start the timer
    yellowLightOn := TRUE;
    redLightOn := FALSE;
    timer(IN := NOT emergencyVehicleApproaching);

ELSE
    // Check the timer and switch to the next light if it's time
    IF timer.Q THEN
        IF greenLightOn THEN
            greenLightOn := FALSE;
            yellowLightOn := TRUE;
            timer(IN := NOT emergencyVehicleApproaching);

        ELSEIF yellowLightOn THEN
            yellowLightOn := FALSE;
            redLightOn := TRUE;
            timer(IN := FALSE);

        ELSE // red light is on
            redLightOn := FALSE;
            greenLightOn := TRUE;
            timer(IN := NOT emergencyVehicleApproaching);
        END_IF;
    END_IF;
END_IF;

// Set the traffic lights based on the variables
SetTrafficLights(greenLightOn, yellowLightOn, redLightOn);

// Wait for a short time before checking again
DELAY 100ms;

END_WHILE

// Read the pedestrian button state from an input FUNCTION ReadPedestrianButton: BOOL // Code to read the input goes here END_FUNCTION

// Read the emergency vehicle sensor state from an input FUNCTION ReadEmergencyVehicleSensor: BOOL // Code to read the input goes here END_FUNCTION

// Set the traffic lights based on the variables PROCEDURE SetTrafficLights( green: BOOL; yellow: BOOL; red: BOOL ) // Code to set the traffic lights goes here END_PROCEDURE

**C-A-R-E:**

🟥 C (Context) – The Problem Setting

You are working on a 61131-3 Structured Text program that controls a traffic light system. The system must handle normal operation, pedestrian crossing requests, and emergency vehicle priority. However, the current implementation includes logical errors and improper PLC programming constructs such as infinite loops (WHILE TRUE DO) and WAIT UNTIL, which are not compliant with standard cyclic execution in PLCs.

⸻

🟩 A (Action) – What Needs to Be Done

To fix the program and make it functionally correct and IEC-compliant:
	•	Replace the infinite loop (WHILE TRUE DO) with cyclic state-driven logic
	•	Remove WAIT UNTIL and instead use state variables and timers to manage transitions
	•	Correctly handle the TON timer without reinitializing it in the same scan cycle
	•	Introduce a state machine with named states like NORMAL, TO_YELLOW, PEDESTRIAN_WAIT, EMERGENCY_OVERRIDE, etc.
	•	Ensure only one mode is active at a time, with clean transitions and recovery back to normal

⸻

🟨 R (Result) – What Will Be Achieved

After refactoring, the program will:
	•	Safely control traffic lights with smooth transitions and prioritized logic
	•	Correctly respond to emergency vehicles by overriding the normal light cycle
	•	Allow pedestrians to cross safely, without logic blocking or collisions
	•	Be cyclic-scan compatible with clean logic separation between input reading, processing, and output setting
	•	Improve readability, maintainability, and extendability for future features (e.g., vehicle counting or night modes)

⸻

🟦 E (Example) – Refactored Logic Overview

Instead of:
WHILE TRUE DO
    WAIT UNTIL ...
    timer(IN := FALSE);
END_WHILE

Use something like:

CASE state OF
    NORMAL:
        IF pedestrianTrig.Q THEN
            state := PEDESTRIAN_WAIT;
        ELSIF emergencySensor THEN
            state := EMERGENCY;
        END_IF

    PEDESTRIAN_WAIT:
        redLight := TRUE;
        pedestrianTimer(IN := TRUE);
        IF pedestrianTimer.Q THEN
            state := TO_GREEN;
        END_IF

    EMERGENCY:
        greenLight := TRUE;
        // Hold this state until emergencySensor = FALSE
END_CASE
