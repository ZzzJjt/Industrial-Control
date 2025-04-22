```
PROGRAM TrafficLightControl
VAR
    pedestrianButtonPressed: BOOL;
    emergencyVehicleApproaching: BOOL;
    greenLightOn: BOOL;
    yellowLightOn: BOOL;
    redLightOn: BOOL;
    timer: TON; // timer to control the duration of green and yellow lights
    pedestrianCrossingTimer: TON; // timer for pedestrian crossing duration
END_VAR

// Initialize the program
greenLightOn := TRUE;
yellowLightOn := FALSE;
redLightOn := FALSE;
timer(IN := NOT emergencyVehicleApproaching, PT := T#5s);
pedestrianCrossingTimer(IN := FALSE, PT := T#10s); // Assume 10 seconds for pedestrian crossing

// Main program loop
WHILE TRUE DO
    // Check for pedestrian button press
    pedestrianButtonPressed := ReadPedestrianButton();
    
    // Check for emergency vehicle approaching
    emergencyVehicleApproaching := ReadEmergencyVehicleSensor();
    
    // Handle emergency vehicles
    IF emergencyVehicleApproaching THEN
        // Turn off all lights and turn on green light for emergency vehicle
        greenLightOn := TRUE;
        yellowLightOn := FALSE;
        redLightOn := FALSE;
        timer(IN := FALSE);
        pedestrianCrossingTimer(IN := FALSE);
        
    ELSEIF pedestrianButtonPressed THEN
        // Turn off all lights and turn on red light and pedestrian light
        greenLightOn := FALSE;
        yellowLightOn := FALSE;
        redLightOn := TRUE;
        
        // Start pedestrian crossing timer
        pedestrianCrossingTimer(IN := TRUE);
        
        // Turn off pedestrian button press once timer starts
        pedestrianButtonPressed := FALSE;
        
        // Reset main timer
        timer(IN := FALSE);
        
    ELSEIF pedestrianCrossingTimer.Q THEN
        // Pedestrian crossing time is over, go to yellow light phase
        yellowLightOn := TRUE;
        redLightOn := FALSE;
        pedestrianCrossingTimer(IN := FALSE);
        
        // Start the timer for yellow light
        timer(IN := TRUE);
        
    ELSEIF timer.Q THEN
        IF greenLightOn THEN
            greenLightOn := FALSE;
            yellowLightOn := TRUE;
            timer(IN := TRUE); // Restart timer for yellow light duration
            
        ELSEIF yellowLightOn THEN
            yellowLightOn := FALSE;
            redLightOn := TRUE;
            timer(IN := FALSE); // Stop timer once red light is on
            
        ELSE // Red light is on
            redLightOn := FALSE;
            greenLightOn := TRUE;
            timer(IN := NOT emergencyVehicleApproaching);
        END_IF;
    END_IF;
    
    // Set the traffic lights based on the variables
    SetTrafficLights(greenLightOn, yellowLightOn, redLightOn);
    
    // Wait for a short time before checking again
    DELAY 100ms;
END_WHILE

// Read the pedestrian button state from an input
FUNCTION ReadPedestrianButton: BOOL
    // Code to read the input goes here
END_FUNCTION

// Read the emergency vehicle sensor state from an input
FUNCTION ReadEmergencyVehicleSensor: BOOL
    // Code to read the input goes here
END_FUNCTION

// Set the traffic lights based on the variables
PROCEDURE SetTrafficLights(
    green: BOOL;
    yellow: BOOL;
    red: BOOL 
)
    // Code to set the traffic lights goes here
END_PROCEDURE
```
