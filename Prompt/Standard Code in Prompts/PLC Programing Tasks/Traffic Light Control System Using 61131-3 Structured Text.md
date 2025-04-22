```
PROGRAM TrafficLightControl
VAR
    // Inputs
    PedestrianButton : BOOL; // Pedestrian push button status
    EmergencyVehicleDetected : BOOL; // Emergency vehicle detection status
    CurrentPhase : INT := 0; // Current phase of the traffic lights (0=Red, 1=Yellow, 2=Green)
    
    // Outputs
    RedLight : BOOL := TRUE; // Red light status
    YellowLight : BOOL := FALSE; // Yellow light status
    GreenLight : BOOL := FALSE; // Green light status
    WalkSignal : BOOL := FALSE; // Walk signal status
    
    // Timers and Counters
    TransitionTimer : TIME := T#0s; // Timer for transitioning between phases
    TransitionDelay : TIME := T#3s; // Delay for transitioning between phases
    GreenDuration : TIME := T#30s; // Duration for green light
    YellowDuration : TIME := T#3s; // Duration for yellow light
    PedestrianWaitTime : TIME := T#15s; // Wait time before pedestrian gets walk signal
    EmergencyClearTime : TIME := T#10s; // Clear time for emergency vehicles
    EmergencyClearTimer : TIME := T#0s; // Timer for emergency vehicle clear time
END_VAR

// Main Control Logic
CASE CurrentPhase OF
    0: // Red phase
        RedLight := TRUE;
        YellowLight := FALSE;
        GreenLight := FALSE;
        WalkSignal := PedestrianButton AND TransitionTimer >= PedestrianWaitTime;
        
        // Emergency vehicle handling
        IF EmergencyVehicleDetected THEN
            EmergencyClearTimer := EmergencyClearTime;
            EmergencyVehicleDetected := FALSE;
        ELSE
            IF TransitionTimer >= GreenDuration THEN
                CurrentPhase := 1; // Transition to Yellow phase
                TransitionTimer := T#0s;
            END_IF;
        END_IF;
        
    1: // Yellow phase
        RedLight := FALSE;
        YellowLight := TRUE;
        GreenLight := FALSE;
        WalkSignal := FALSE;
        
        IF TransitionTimer >= YellowDuration THEN
            CurrentPhase := 2; // Transition to Green phase
            TransitionTimer := T#0s;
        END_IF;
        
    2: // Green phase
        RedLight := FALSE;
        YellowLight := FALSE;
        GreenLight := TRUE;
        WalkSignal := FALSE;
        
        IF TransitionTimer >= GreenDuration THEN
            CurrentPhase := 0; // Transition to Red phase
            TransitionTimer := T#0s;
        END_IF;
        
        // Emergency vehicle handling
        IF EmergencyVehicleDetected THEN
            EmergencyClearTimer := EmergencyClearTime;
            EmergencyVehicleDetected := FALSE;
        END_IF;
        
END_CASE;

// Emergency Vehicle Handling
IF EmergencyClearTimer > T#0s THEN
    RedLight := TRUE;
    YellowLight := FALSE;
    GreenLight := FALSE;
    WalkSignal := FALSE;
    EmergencyClearTimer := EmergencyClearTimer - T#1s;
END_IF;

// Increment Transition Timer
IF TCU(T#1s) THEN
    IF CurrentPhase = 1 OR CurrentPhase = 2 THEN
        TransitionTimer := TransitionTimer + T#1s;
    ELSE
        IF PedestrianButton THEN
            TransitionTimer := TransitionTimer + T#1s;
        END_IF;
    END_IF;
END_IF;

// Debugging outputs (for simulation purposes)
// Example: Write("Current Phase", CurrentPhase);
// Example: Write("Red Light", RedLight);
// Example: Write("Yellow Light", YellowLight);
// Example: Write("Green Light", GreenLight);
// Example: Write("Walk Signal", WalkSignal);
// Example: Write("Transition Timer", TransitionTimer);
// Example: Write("Emergency Clear Timer", EmergencyClearTimer);

END_PROGRAM
```
