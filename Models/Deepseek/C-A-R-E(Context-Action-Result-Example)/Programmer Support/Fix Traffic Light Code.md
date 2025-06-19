PROGRAM TrafficLightControlSystem
VAR_INPUT
    pedestrianRequest : BOOL; // Pedestrian crossing request input
    emergencyVehicle : BOOL;  // Emergency vehicle presence input
END_VAR

VAR_OUTPUT
    redLight : BOOL;           // Red light output
    yellowLight : BOOL;         // Yellow light output
    greenLight : BOOL;         // Green light output
    pedestrianSignal : BOOL;   // Pedestrian signal output
END_VAR

VAR
    state : INT := 0;          // State variable for state machine
    pedestrianTimer : TON;     // Timer for pedestrian crossing
    normalCycleTimer : TON;    // Timer for normal cycle
    emergencyOverrideTimer : TON; // Timer for emergency override
    pedestrianDuration : TIME := T#5s; // Duration for pedestrian crossing
    normalGreenDuration : TIME := T#4s; // Duration for green light in normal cycle
    normalYellowDuration : TIME := T#2s; // Duration for yellow light in normal cycle
    emergencyGreenDuration : TIME := T#3s; // Duration for green light in emergency mode
    dt : TIME := T#100ms;      // Sampling time for cyclic execution
END_VAR

// Initialize timers
pedestrianTimer(IN := FALSE);
normalCycleTimer(IN := FALSE);
emergencyOverrideTimer(IN := FALSE);

// State constants
CONSTANT
    STATE_NORMAL : INT := 0;
    STATE_TO_YELLOW : INT := 1;
    STATE_PEDESTRIAN_WAIT : INT := 2;
    STATE_EMERGENCY_OVERRIDE : INT := 3;

// Main cyclic execution loop
CASE state OF
    STATE_NORMAL:
        // Normal traffic light cycle: GREEN -> YELLOW -> RED
        IF emergencyVehicle THEN
            state := STATE_EMERGENCY_OVERRIDE;
        ELSIF pedestrianRequest THEN
            state := STATE_TO_YELLOW;
        ELSE
            // Start or continue normal cycle timer
            normalCycleTimer(IN := TRUE);
            IF normalCycleTimer.Q THEN
                // Transition to yellow after green duration
                normalCycleTimer(IN := FALSE); // Reset timer
                state := STATE_TO_YELLOW;
            END_IF;
        END_IF;

        // Set outputs for normal green phase
        redLight := FALSE;
        yellowLight := FALSE;
        greenLight := TRUE;
        pedestrianSignal := FALSE;

    STATE_TO_YELLOW:
        // Transition from green to yellow
        yellowLight := TRUE;
        greenLight := FALSE;

        // Start or continue transition timer
        normalCycleTimer(IN := TRUE);
        IF normalCycleTimer.Q THEN
            // Transition to red after yellow duration
            normalCycleTimer(IN := FALSE); // Reset timer
            state := STATE_NORMAL; // Restart normal cycle
        END_IF;

        pedestrianSignal := FALSE;

    STATE_PEDESTRIAN_WAIT:
        // Pedestrian crossing request handling
        redLight := TRUE;
        yellowLight := FALSE;
        greenLight := FALSE;
        pedestrianSignal := TRUE;

        // Start or continue pedestrian timer
        pedestrianTimer(IN := TRUE);
        IF pedestrianTimer.Q THEN
            // Transition back to normal cycle after pedestrian duration
            pedestrianTimer(IN := FALSE); // Reset timer
            state := STATE_NORMAL;
        END_IF;

    STATE_EMERGENCY_OVERRIDE:
        // Emergency vehicle priority
        redLight := FALSE;
        yellowLight := FALSE;
        greenLight := TRUE;
        pedestrianSignal := FALSE;

        // Continue emergency override until no emergency vehicle
        IF NOT emergencyVehicle THEN
            state := STATE_NORMAL;
        END_IF;

    ELSE
        // Default case to handle unexpected states
        state := STATE_NORMAL;
END_CASE;

// Inline comments explaining the logic:
// The program implements a state machine to control traffic lights with smooth transitions and prioritized logic.
// States include NORMAL, TO_YELLOW, PEDESTRIAN_WAIT, and EMERGENCY_OVERRIDE.
// The program handles pedestrian crossing requests and emergency vehicle priorities without blocking logic.
// Timers manage transitions between states, ensuring cyclic-scan compatibility.
// Outputs are set based on the current state, maintaining clear separation between input reading, processing, and output setting.



