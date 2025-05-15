Certainly! Below is an IEC 61131-3 Structured Text program that implements the B.2 Reaction step of an adhesive production batch control recipe according to ISA-88 standards. The program includes modular methods for heating, mixing, and holding the reaction environment, along with logic for transitioning between sub-steps using timers and real-time conditions.

### Adhesive Production Batch Control Recipe - B.2 Reaction Step

```st
PROGRAM ReactionStep
VAR
    // Process Parameters
    targetTemperature : REAL := 85.0; // Target temperature in °C
    mixingSpeed : INT := 600;         // Mixing speed in RPM
    holdTime : TIME := T#20m;         // Hold time in minutes

    // Sensors and Actuators
    currentTemperature : REAL;        // Current temperature sensor reading
    mixerRunning : BOOL := FALSE;     // Mixer running status

    // Timers
    heatTimer : TON;                  // Timer for heating
    mixTimer : TON;                   // Timer for mixing
    holdTimer : TON;                  // Timer for holding

    // Control Variables
    currentState : INT := 0;          // State: 0 = Idle, 1 = Heating, 2 = Mixing, 3 = Holding, 4 = Complete
    heatStable : BOOL := FALSE;       // Flag to indicate stable heating
END_VAR

// Methods
METHOD StartHeating(temp : REAL)
    targetTemperature := temp;
    heatTimer(IN := TRUE, PT := T#5m); // Assume 5 minutes for heating to stabilize
END_METHOD

METHOD StartMixing(speed : INT)
    mixingSpeed := speed;
    mixerRunning := TRUE;
    mixTimer(IN := TRUE, PT := T#10s); // Assume 10 seconds for mixer startup
END_METHOD

METHOD HoldReaction(time : TIME)
    holdTime := time;
    holdTimer(IN := TRUE, PT := holdTime);
END_METHOD

METHOD StopMixing()
    mixerRunning := FALSE;
END_METHOD

// Main cyclic execution
CASE currentState OF
    0: // Idle State
        IF startCommand THEN
            currentState := 1; // Transition to Heating State
            StartHeating(targetTemperature);
        END_IF;

    1: // Heating State
        // Check if temperature has reached the target and stabilized
        IF currentTemperature >= targetTemperature AND NOT heatTimer.Q THEN
            heatStable := TRUE;
        END_IF;

        IF heatStable AND heatTimer.Q THEN
            currentState := 2; // Transition to Mixing State
            StartMixing(mixingSpeed);
            heatTimer(IN := FALSE); // Reset timer
        END_IF;

    2: // Mixing State
        // Wait for mixer to start and stabilize
        IF mixerRunning AND mixTimer.Q THEN
            currentState := 3; // Transition to Holding State
            HoldReaction(holdTime);
            mixTimer(IN := FALSE); // Reset timer
        END_IF;

    3: // Holding State
        // Hold reaction for specified duration
        IF holdTimer.Q THEN
            currentState := 4; // Transition to Complete State
            StopMixing();
            holdTimer(IN := FALSE); // Reset timer
        END_IF;

    4: // Complete State
        // End of reaction step
        currentState := 0; // Return to Idle State
        stopCommand := FALSE; // Reset stop command

    ELSE:
        // Handle unexpected states
        currentState := 0; // Reset to Idle State
END_CASE;
END_PROGRAM
```

### Explanation of the Program

#### Variable Declarations
- **Process Parameters**: Constants for target temperature, mixing speed, and hold time.
- **Sensors and Actuators**: Variables to simulate current temperature and mixer status.
- **Timers**: `TON` timers for heating, mixing, and holding phases.
- **Control Variables**: Flags and variables to track the current state and process conditions.

#### Methods
- **StartHeating(temp : REAL)**: Initiates heating with a stabilization period.
- **StartMixing(speed : INT)**: Starts the mixer at the specified speed.
- **HoldReaction(time : TIME)**: Holds the reaction for a specified duration.
- **StopMixing()**: Stops the mixer.

#### Main Cyclic Execution
- **Idle State (0)**: Waits for a start command to transition to the heating state.
- **Heating State (1)**:
  - Checks if the temperature has reached and stabilized at the target value.
  - Transitions to the mixing state once heating is complete.
- **Mixing State (2)**:
  - Waits for the mixer to start and stabilize.
  - Transitions to the holding state once mixing is complete.
- **Holding State (3)**:
  - Holds the reaction for the specified duration.
  - Transitions to the complete state once holding is complete.
- **Complete State (4)**:
  - Ends the reaction step and returns to the idle state.
  - Resets the stop command.

### Key Features
- **Modularity**: Each phase (heating, mixing, holding) is implemented as a separate method.
- **Reusability**: Methods can be reused across different batch steps or processes.
- **Real-Time Monitoring**: Uses sensors and actuators to monitor and control process conditions.
- **Safe Sequencing**: Ensures strict adherence to the recipe logic with accurate parameter tracking and safe sequencing of operations.

This program reflects ISA-88 principles by being modular, maintainable, and reusable, ensuring reliable execution of the batch step with accurate parameter tracking and safe sequencing of operations.
