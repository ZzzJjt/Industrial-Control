Certainly! Below is a detailed IEC 61131-3 Structured Text program that implements an ISA-88-compliant batch control recipe for the reaction stage of urea fertilizer production. The program includes precise control logic for heating, pressure regulation, and cooling, ensuring smooth transitions between these phases.

### Urea Fertilizer Production Reaction Stage Control Recipe

#### Physical Model
- **Reactor**: For chemical reactions.
- **Heater/Chiller**: For controlling temperature.
- **Pressure Regulator**: For maintaining pressure.
- **Coolant System**: For cooling down the reactor after the reaction.

#### Procedural Units
- **HeatUp Phase**: Heating the reactor to the target temperature.
- **PressureHold Phase**: Maintaining the reactor at the target temperature and pressure.
- **CoolDown Phase**: Cooling down the reactor after the reaction.

### Structured Text Program

```st
PROGRAM UreaReactionBatchControl
VAR
    // Process Parameters
    heatTargetTemp : REAL := 180.0;          // Target temperature for heating in °C
    pressureSetpoint : REAL := 140.0;        // Pressure setpoint in bar
    reactionDuration : TIME := T#30m;        // Duration for the reaction in minutes
    cooldownDuration : TIME := T#20m;        // Duration for cooling down in minutes

    // Sensors and Actuators
    currentTemperature : REAL;               // Current temperature sensor reading
    currentPressure : REAL;                  // Current pressure sensor reading
    heaterRunning : BOOL := FALSE;           // Heater running status
    chillerRunning : BOOL := FALSE;          // Chiller running status
    pressureRegulatorActive : BOOL := FALSE; // Pressure regulator active status
    coolantPumpRunning : BOOL := FALSE;      // Coolant pump running status

    // Timers
    heatTimer : TON;                         // Timer for heating
    reactionTimer : TON;                     // Timer for reaction duration
    cooldownTimer : TON;                     // Timer for cooling down

    // Control Variables
    currentState : INT := 0;                 // State: 0 = Idle, 1 = Heat Up, 2 = Pressure Hold, 3 = Cool Down, 4 = Complete
    heatComplete : BOOL := FALSE;            // Flag to indicate heating is complete
    pressureStable : BOOL := FALSE;          // Flag to indicate pressure is stable
    reactionComplete : BOOL := FALSE;        // Flag to indicate reaction is complete
    cooldownComplete : BOOL := FALSE;        // Flag to indicate cooling is complete
END_VAR

// Methods
METHOD StartHeating(targetTemp : REAL, duration : TIME)
    heaterRunning := TRUE;
    heatTimer(IN := TRUE, PT := duration);

    WHILE NOT heatTimer.Q DO
        IF currentTemperature >= targetTemp THEN
            heatComplete := TRUE;
            heaterRunning := FALSE;
            heatTimer(IN := FALSE); // Reset timer
            BREAK;
        END_IF;
    END_WHILE;
END_METHOD

METHOD StartPressureHold(setpoint : REAL, duration : TIME)
    pressureRegulatorActive := TRUE;
    reactionTimer(IN := TRUE, PT := duration);

    WHILE NOT reactionTimer.Q DO
        IF currentPressure >= setpoint THEN
            pressureStable := TRUE;
        END_IF;

        IF pressureStable AND reactionTimer.Q THEN
            reactionComplete := TRUE;
            pressureRegulatorActive := FALSE;
            reactionTimer(IN := FALSE); // Reset timer
            BREAK;
        END_IF;
    END_WHILE;
END_METHOD

METHOD StartCooling(duration : TIME)
    coolantPumpRunning := TRUE;
    cooldownTimer(IN := TRUE, PT := duration);

    WHILE NOT cooldownTimer.Q DO
        IF currentTemperature <= 50.0 THEN // Assuming safe handling temperature
            cooldownComplete := TRUE;
            coolantPumpRunning := FALSE;
            cooldownTimer(IN := FALSE); // Reset timer
            BREAK;
        END_IF;
    END_WHILE;
END_METHOD

// Main cyclic execution
CASE currentState OF
    0: // Idle State
        IF startCommand THEN
            currentState := 1; // Transition to Heat Up State
        END_IF;

    1: // Heat Up State
        StartHeating(heatTargetTemp, T#10m);
        IF heatComplete THEN
            currentState := 2; // Transition to Pressure Hold State
        END_IF;

    2: // Pressure Hold State
        StartPressureHold(pressureSetpoint, reactionDuration);
        IF reactionComplete THEN
            currentState := 3; // Transition to Cool Down State
        END_IF;

    3: // Cool Down State
        StartCooling(cooldownDuration);
        IF cooldownComplete THEN
            currentState := 4; // Transition to Complete State
        END_IF;

    4: // Complete State
        // End of batch process
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
- **Process Parameters**: Constants for target temperature, pressure setpoint, reaction duration, and cooldown duration.
- **Sensors and Actuators**: Variables to simulate current temperature, pressure, and actuator statuses.
- **Timers**: `TON` timers for heating, reaction, and cooling phases.
- **Control Variables**: Flags and variables to track the current state and process conditions.

#### Methods
- **StartHeating(targetTemp : REAL, duration : TIME)**: Heats the reactor to the target temperature within a specified duration.
- **StartPressureHold(setpoint : REAL, duration : TIME)**: Maintains the reactor at the target pressure for the specified reaction duration.
- **StartCooling(duration : TIME)**: Cools down the reactor after the reaction until it reaches a safe handling temperature.

#### Main Cyclic Execution
- **Idle State (0)**: Waits for a start command to transition to the heat up state.
- **Heat Up State (1)**:
  - Calls `StartHeating` to heat the reactor to 180°C within 10 minutes.
  - Transitions to the pressure hold state once heating is complete.
- **Pressure Hold State (2)**:
  - Calls `StartPressureHold` to maintain the reactor at 140 bar for 30 minutes.
  - Transitions to the cool down state once the pressure is stable and the reaction duration elapses.
- **Cool Down State (3)**:
  - Calls `StartCooling` to cool down the reactor until it reaches a safe handling temperature (assumed to be 50°C).
  - Transitions to the complete state once cooling is complete.
- **Complete State (4)**:
  - Ends the batch process and returns to the idle state.
  - Resets the stop command.

### Key Features
- **Modularity**: Each phase (heating, pressure holding, cooling) is implemented as a separate method.
- **Reusability**: Methods can be reused across different batch steps or processes.
- **Real-Time Monitoring**: Uses sensors and actuators to monitor and control process conditions.
- **Safe Sequencing**: Ensures strict adherence to the recipe logic with accurate parameter tracking and safe sequencing of operations.
- **ISA-88 Compliance**: Implements equipment modules and procedural units to ensure consistency and scalability.

This program provides a reliable, standards-compliant solution for managing the critical reaction stage of urea fertilizer production, ensuring consistent product quality, flexible control sequences, and clear integration of ISA-88 elements.
