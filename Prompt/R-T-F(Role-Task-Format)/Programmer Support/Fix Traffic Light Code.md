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

